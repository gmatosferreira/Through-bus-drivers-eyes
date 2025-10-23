# Events: Bus Bunching


``` r
library(dplyr)
library(tidyverse)
```

Build table for all trips for period

``` r
gtfs = tidytransit::read_gtfs("https://files.mobilitydatabase.org/mdb-1032/mdb-1032-202504070123/mdb-1032-202504070123.zip")
bus_routes = gtfs_unified$routes |> filter(!grepl("E", route_short_name))
gtfs_bus = gtfstools::filter_by_route_id(gtfs, route_id = bus_routes$route_id, keep = TRUE)
gtfs = tidytransit::as_tidygtfs(gtfs_bus)

# Trips extend with info
# > Add stop_times departure and arrival stop
trips_extended = gtfs$trips |> 
  left_join(gtfs$stop_times, by = "trip_id") |>
  arrange(trip_id, stop_sequence) |>
  group_by(trip_id) |> 
  summarise(
    route_id = first(route_id),
    trip_id = first(trip_id),
    service_id = first(service_id),
    direction_id = first(direction_id),
    shape_id = first(shape_id),
    departure_stop = first(stop_id),
    departure_stop_seq = first(stop_sequence),
    departure_time = first(departure_time),
    arrival_stop = last(stop_id),
    arrival_stop_seq = last(stop_sequence),
    arrival_time = last(arrival_time),
    duration = as.integer(difftime(
      as.POSIXct(arrival_time, format="%H:%M:%S"), 
      as.POSIXct(departure_time, format="%H:%M:%S"), 
      units = "mins"
    ))
  )
# > Associate route number
trips_extended = trips_extended |> 
  left_join(gtfs$routes |> select(route_id, route_short_name), by="route_id")
# > Add voyage number (trip number for that route on that day)
trips_extended <- trips_extended %>%
  arrange(service_id, route_short_name, departure_time) %>%
  group_by(service_id, route_short_name) %>%
  mutate(trip_number = row_number()) %>%
  ungroup()

# Iterate interval
trips_with_day = data.frame()
lapply(dates, function (date) {
  message(sprintf("Computing for date: %s", ymd(date)))
  gtfs_date = suppressWarnings({ tidytransit::filter_feed_by_date(gtfs, ymd(date)) })
  services = unique(gtfs_date$calendar$service_id)
  trips_with_day_date = trips_extended |> filter(service_id %in% services)
  trips_with_day_date$date = date
  trips_with_day <<- rbind(trips_with_day, trips_with_day_date)
})
```

Load events

``` r
events_11 = read.csv("csv/events_20250501_20250531_bus_categorised_category_11.csv") # Using stop detection events (ID 11)
```

Clean data

``` r
routes_stops = trips_with_day |> 
  dplyr::select(route_short_name, departure_stop_id, arrival_stop_id) |> 
  distinct() |> 
  pivot_longer(
    cols = c(departure_stop_id, arrival_stop_id),
    names_to = "stop_type",
    values_to = "stop_ref"
  )

stops_names = events_11 |> 
  select(BusStopNumber, stop_ref) |> 
  distinct()

# Remove edge stations for each route
events_11 = events_11 |> 
  left_join(
    routes_stops |> select(route_short_name, stop_ref) |> mutate(terminal_stop = TRUE), 
    by = c(
      "RouteNumber" = "route_short_name",
      "BusStopNumber" = "stop_ref"
      ),
    multiple = "first"
    ) |>
  filter(is.na(terminal_stop))

# Remove stops that are not on the route
gtfs_routes_stops = gtfs_04_05$routes |>
  select(route_id, route_short_name) |>
  left_join(
    gtfs_04_05$trips |> select(route_id, trip_id) |> distinct(),
    by = "route_id"
  ) |> left_join(
    gtfs_04_05$stop_times |> select(trip_id, stop_id, stop_sequence),
    by = "trip_id"
  ) |> 
  select(route_short_name, stop_id) |> distinct() |>
  mutate(stop_id = as.integer(stop_id)) |> 
  left_join(stops_names |> mutate(BusStopNumber = as.integer(BusStopNumber)), by = c("stop_id" = "BusStopNumber"))

events_11 = events_11 |> 
  left_join(
    gtfs_routes_stops |> select(route_short_name, stop_id) |> mutate(part_of_route = TRUE), 
    by = c(
      "RouteNumber" = "route_short_name",
      "BusStopNumber" = "stop_id"
      ),
    multiple = "first"
    ) |> filter(part_of_route == TRUE)
```

Evaluate bus bunching

``` r
# % of stop times in which more than one bus arrived in the -1 and +5 minutes window
bunching = events_11 |> 
  group_by(BusStopNumber, RouteNumber, direction) |> 
  arrange(timestamp, .by_group = TRUE) |>
  mutate(
    timestamp = as.POSIXct(timestamp / 1000, origin = "1970-01-01", tz = "UTC"), 
    date = as.Date(timestamp),
    hh = as.integer(format(timestamp, "%H"))
  ) |>
  mutate(
    # difference from the *previous* detection (mins)
    diff_prev = as.numeric(difftime(timestamp, lag(timestamp), units = "mins")),
    # difference to the *next* detection (mins)
    diff_next = as.numeric(difftime(lead(timestamp), timestamp, units = "mins")),
    
    # bunching if previous is within -1 min, or next within +5 mins
    bunched = (diff_prev >= -1 & diff_prev <= 0) |
              (diff_next >= 0 & diff_next <= 5)
  ) %>%
  ungroup()

bunching_summary = bunching |> 
  summarise(
    n_stops = n(),
    n_bunched = sum(bunched, na.rm = TRUE),
    pct_bunched = n_bunched / n_stops * 100
  ) |> 
  ungroup() |> 
  arrange(desc(pct_bunched)) |>
  as.vector()

bunching_summary_per_route_AND_stop = bunching |>
  group_by(BusStopNumber, RouteNumber) |> 
  summarise(
    n_stops = n(),
    n_bunched = sum(bunched, na.rm = TRUE),
    pct_bunched = n_bunched / n_stops * 100
  ) |> 
  ungroup() |> 
  arrange(desc(pct_bunched)) |> 
  left_join(stops_names, by = "BusStopNumber")
```
