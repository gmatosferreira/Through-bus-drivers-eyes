# Events: Reliability Buffer Index


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

Load data

``` r
trips = read.csv("csv/events_20250501_20250531_trips_processed.csv")
```

Compute RBI for planned trips

``` r
reliability_planned_by_route = trips_with_day |> 
  filter(date == "2025-05-07") |> 
  group_by(route_short_name, departure_stop, arrival_stop) |> 
  summarise(
    mean_duration = mean(duration, na.rm = TRUE),
    p_95_duration = quantile(duration, 0.95, na.rm = TRUE),
    reliability_buffer_index = (p_95_duration - mean_duration)/mean_duration * 100
  )
```

Compute RBI for executed trips and relate with planned

``` r
trips = trips |> 
  filter(!is.na(trip_id) & trip_id!="") |> 
  mutate(
    hh = as.numeric(format(as.POSIXct(start_timestamp/1000, origin="1970-01-01", tz="Europe/Lisbon"), "%H"))
  )

reliability_global = trips |> 
  ungroup() |>
  summarise(
    mean_duration = mean(duration, na.rm = TRUE),
    p_95_duration = quantile(duration, 0.95, na.rm = TRUE),
    reliability_buffer_index = (p_95_duration - mean_duration)/mean_duration * 100
  ) |> as.vector()

reliability_global_by_route = trips |> 
  ungroup() |>
  group_by(RouteNumber, start_stop_id, end_stop_id) |>
  summarise(
    n_trips = n(),
    mean_duration = mean(duration, na.rm = TRUE),
    p_95_duration = quantile(duration, 0.95, na.rm = TRUE),
    reliability_buffer_index = (p_95_duration - mean_duration)/mean_duration * 100,
    reliability_buffer_index_times_n = reliability_buffer_index * n_trips
  ) |> left_join(
    reliability_planned_by_route |> # ATTENTION! From 3.1.2. GTFS stats.Rmd
      select(route_short_name, reliability_buffer_index, departure_stop, arrival_stop, mean_duration, p_95_duration) |>
      rename(mean_duration_planned = mean_duration, p_95_duration_planned = p_95_duration),
    by = c(
      "RouteNumber" = "route_short_name",
      "start_stop_id" = "departure_stop",
      "end_stop_id" = "arrival_stop"
    ),
    suffix = c("_real", "_planned")
  ) |> mutate(
    difference_real_planned = reliability_buffer_index_real - reliability_buffer_index_planned,
    reliability_buffer_index_planned_times_n = reliability_buffer_index_planned * n_trips
  ) |> arrange(desc(difference_real_planned))
```
