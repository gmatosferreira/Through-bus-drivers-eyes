# Categorizing Free Text Messages


``` r
library(tm)
library(NLP)
library(stringr)
library(topicmodels)
library(tidyverse)
library(tidytext)
library(ggplot2)
library(readxl)
library(SnowballC)
library(textstem)
library(skimr)
library(kableExtra)
library(dplyr)
library(quanteda)
library(igraph)
library(ggraph)
library(tidyr)
library(broom)
```

### Data Cleaning

Data load, assuming the data is in a CSV file named
“messages_2025_freeText.csv”, with columns “RouteNumber” and
“MessageText”.

``` r
messages_freeText = read.csv("csv/messages_2025_freeText.csv")
messages_freeText =  messages_freeText |> filter(!grepl("E", RouteNumber)) # Consider only bus routes (remove trams)
```

Lemmatization

``` r
library(lemmar) # remotes::install_github("trinker/lemmar") 

text <- messages_freeText[,c("MessageText")]
text <- gsub(pattern = "\\W", replace = " ", text) # Remove punctuation
text2 <- gsub(pattern = "\\d", replace = " ", text) # Remove Numbers (digits)
text3 <- tolower(text2) # Lowercase words
text4 <- gsub(pattern = "\\b[A-z]\\b{1}", replace = " ", text3) # Remove single words 
text5 <- stripWhitespace(text4) # Remove unnecessary whitespaces

# Context adjustments (remove some lemmatizations that should be kept and add custom ones)
hash_lemma_pt_carris = hash_lemma_pt
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="obrigado", ] # remove "obrigado" as it is not a lemma
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="estado", ] # to avoid "estado" being lemmatized to "estar" (pedido de estado)
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="pedido", ] # to avoid "pedido" being lemmatized to "pedir" (pedido de estado)
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="reservado", ] # to avoid "reservado" being lemmatized to "reservar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="cedo", ] # to avoid "cedo" being lemmatized to "ceder"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="caminho", ] # to avoid "caminho" being lemmatized to "caminhar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="livre", ] # to avoid "livre" being lemmatized to "livrar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="centro", ] # to avoid "centro" being lemmatized to "centrar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="passe", ] # to avoid "passe" being lemmatized to "passar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="sentido", ] # to avoid "sentido" being lemmatized to "sentir"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="entre", ] # to avoid "entre" being lemmatized to "entrar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="esperança", ] # to avoid "esperança" being lemmatized to "esperançar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="rua", ] # to avoid "rua" being lemmatized to "ruir"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="rato", ] # to avoid "Rato" being lemmatized to "ratar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="tarde", ] # to avoid "tarde" being lemmatized to "tardar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="carteira", ] # to avoid "carteira" being lemmatized to "carteiro"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="chapa", ] # to avoid "chapa" being lemmatized to "chapar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="sueste", ] # to avoid "sueste" being lemmatized to "suestar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="ajuda", ] # to avoid "Ajuda" being lemmatized to "ajudar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="hora", ] # to avoid "hora" being lemmatized to "horar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="oriente", ] # to avoid "Oriente" being lemmatized to "orientar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="pelo", ] # to avoid "pelo" being lemmatized to "pelar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="sapadores", ] # to avoid "Sapadores" being lemmatized to "sapador"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="amoreiras", ] # to avoid "Amoreiras" being lemmatized to "amoreira"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="amaro", ] # to avoid "Amaro" being lemmatized to "amarar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="corte", ] # to avoid "corte" being lemmatized to "cortar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="bispo", ] # to avoid "Bispo" being lemmatized to "bispar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="luz", ] # to avoid "Luz" being lemmatized to "luzir"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="consola", ] # to avoid "consola" being lemmatized to "consolar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="restelo", ] # to avoid "Restelo" being lemmatized to "restelar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="saco", ] # to avoid "saco" being lemmatized to "sacar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="pela", ] # to avoid "pela" being lemmatized to "pelar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="velha", ] # to avoid "Velha" being lemmatized to "velho"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="mala", ] # to avoid "mala" being lemmatized to "malo"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="cuidado", ] # to avoid "cuidado" being lemmatized to "cuidar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="marques", ] # to avoid "marques" being lemmatized to "marcar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="campo", ] # to avoid "Campo" being lemmatized to "campar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="comercio", ] # to avoid "Comercio" being lemmatized to "comerciar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="buraca", ] # to avoid "buraca" being lemmatized to "Buracar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="piso", ] # to avoid "piso" being lemmatized to "pisar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="alameda", ] # to avoid "Alameda" being lemmatized to "alamedar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="silva", ] # to avoid "Silva" being lemmatized to "silvar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="arco", ] # to avoid "Arco" being lemmatized to "arcar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="cego", ] # to avoid "Cego" being lemmatized to "cegar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="picheleira", ] # to avoid "Picheleira" being lemmatized to "picheleiro"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="torre", ] # to avoid "Torre" being lemmatized to "torrar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="como", ] # to avoid "como" being lemmatized to "comer"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="transito", ] # to avoid "transito" being lemmatized to "transitar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="conforme", ] # to avoid "conforme" being lemmatized to "conformar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="bordo", ] # to avoid "bordo" being lemmatized to "bordar"
hash_lemma_pt_carris = hash_lemma_pt_carris[c(token)!="rosa", ] # to avoid "rosa" being lemmatized to "rosar"

hash_lemma_pt_carris = rbind(
  hash_lemma_pt_carris,
  data.frame(token="obg", lemma="obrigado"),
  data.frame(token="obr", lemma="obrigado"),
  data.frame(token="obrg", lemma="obrigado"),
  data.frame(token="obrig", lemma="obrigado"),
  data.frame(token="obgdo", lemma="obrigado"),
  data.frame(token="obgd", lemma="obrigado"),
  data.frame(token="mot", lemma="motorista"),
  data.frame(token="sr", lemma="senhor"),
  data.frame(token="info", lemma="informar"),
  data.frame(token="sentidos", lemma="sentido"),
  data.frame(token="av", lemma="avenida"),
  data.frame(token="min", lemma="minuto"),
  data.frame(token="mts", lemma="minuto"),
  data.frame(token="reserv", lemma="reservado"),
  data.frame(token="res", lemma="reservado"),
  data.frame(token="est", lemma="estação"),
  data.frame(token="pç", lemma="praça"),
  data.frame(token="circ", lemma="circular"),
  data.frame(token="cid", lemma="cidade"),
  data.frame(token="univ", lemma="universitária"),
  data.frame(token="universitaria", lemma="universitária"),
  data.frame(token="lg", lemma="largo"),
  data.frame(token="mq", lemma="marquês"),
  data.frame(token="marques", lemma="marquês"),
  data.frame(token="contate", lemma="contactar"),
  data.frame(token="contatar", lemma="contactar"),
  data.frame(token="contac", lemma="contactar"),
  data.frame(token="contact", lemma="contactar"),
  data.frame(token="cont", lemma="contactar"),
  data.frame(token="col", lemma="colégio"),
  data.frame(token="va", lemma="ir"),
  data.frame(token="cp", lemma="campo"),
  data.frame(token="bº", lemma="bairro"),
  data.frame(token="alcantara", lemma="alcântara"),
  data.frame(token="possivel", lemma="possível"),
  data.frame(token="atençao", lemma="atenção"),
  data.frame(token="belem", lemma="belém"),
  data.frame(token="estefania", lemma="estefânia"),
  data.frame(token="ausencia", lemma="ausência"),
  data.frame(token="pass", lemma="passageiro"),
  data.frame(token="ch", lemma="chapa"),
  data.frame(token="transito", lemma="trânsito"),
  data.frame(token="bandeira", lemma="bandeiras"),
  data.frame(token="band", lemma="bandeiras"),
  data.frame(token="econtrou", lemma="encontrar"),
  data.frame(token="encontou", lemma="encontrar")
)
text6 <- lemmatize_strings(text5, dictionary = hash_lemma_pt_carris)

text_cleaned <- data.frame(text6)
colnames(text_cleaned) <- c("message")
messages_freeText$message_cleaned = text_cleaned$message
messages_freeText = messages_freeText |> filter(stripWhitespace(message_cleaned) != "") # Remove empty messages
```

Stop words

``` r
# Start with base list
stop_words_pt = read.csv("https://raw.githubusercontent.com/stopwords-iso/stopwords-pt/master/stopwords-pt.txt")

# Context adjustments
stop_words_pt = stop_words_pt |> filter(word != "estado")

# Extend with personal names
pt_countries = c("PT", "BR", "AO", "CV", "GW", "GQ", "MZ", "ST", "TL") # ISO
first_names = read.csv("https://raw.githubusercontent.com/sigpwned/popular-names-by-country-dataset/refs/heads/main/common-forenames-by-country.csv") |> 
  filter(Country %in% pt_countries) |>  
  dplyr::select(Localized.Name)
colnames(first_names) = c("word")
first_names = first_names |> add_row(word="sónia")

last_names = read.csv("https://raw.githubusercontent.com/sigpwned/popular-names-by-country-dataset/refs/heads/main/common-surnames-by-country.csv") |> 
  filter(Country %in% pt_countries) |> 
  dplyr::select(Localized.Name)
colnames(last_names) = c("word")
last_names = last_names |> add_row(word="covelinhas")

names = rbind(first_names, last_names)
names$word = tolower(names$word)

stop_words_pt = rbind(stop_words_pt, names)

# Extend with stop names
library(GTFShift)
gtfs = GTFShift::load_feed("https://gateway.carris.pt/gateway/gtfs/api/v2.8/GTFS")

stop_names = gtfs$stops$stop_name
stop_names <- gsub(pattern = "\\W", replace = " ", stop_names) # Remove punctuation
stop_names <- gsub(pattern = "\\d", replace = " ", stop_names) # Remove Numbers (digits)
stop_names <- tolower(stop_names) # Lowercase words
stop_names <- gsub(pattern = "\\b[A-z]\\b{1}", replace = " ", stop_names)  # Remove single words 
stop_names <- stripWhitespace(stop_names) # Remove unnecessary whitespace (e.g., multiple spaces)
stop_names = append(stop_names, c("avenida", "rua", "praça", "parque", "quinta", "alameda", "marginal")) # Extend with common abreviated names
stop_names = unique(unlist(strsplit(stop_names, "\\s+"))) # Separate names into words

stop_names = stop_names[unlist(lapply(stop_names, function(x) !(x %in% c("estado", "terminal"))))] # Remove with words that are also operational related nouns and are relevant for analysis

stop_names = data.frame(stop_names)
colnames(stop_names) = c("word")

stop_words_pt = rbind(stop_words_pt, stop_names)
```

### Topic Modelling

``` r
# Anonymize messages
text_cleaned_tm = removeWords(messages_freeText$message_cleaned, stop_names$word)
text_cleaned_tm = removeWords(text_cleaned_tm, names$word)
text_cleaned_tm = removeWords(text_cleaned_tm, c("obrigado"))
text_cleaned_tm  = Filter(function(x) trimws(x) != "", text_cleaned_tm)

# Create a corpus
corpus <- Corpus(VectorSource(text_cleaned_tm))

# Create Document-Term Matrix
dtm <- DocumentTermMatrix(corpus) 
dtm <- dtm[slam::row_sums(dtm) > 0, ] # Remove empty documents (rows with zero sums)

# Run LDA using Gibbs sampling
k <- 12
ldaOut <- LDA(dtm, k, method="Gibbs", control=list(seed = 42)) 
lda_topics <- ldaOut %>% tidy(matrix = "beta") %>% arrange(desc(beta))

# Select X most frequent terms in each topic, with X the P75 number of words per message
n_words <- quantile(text_cleaned$n_words, 0.75)
word_probs <- lda_topics %>%
  group_by(topic) %>%
  top_n(n_words, beta) %>%
  ungroup() %>%
  #Create term2, a factor ordered by word probability
  mutate(term2 = fct_reorder(term, beta))

# Plot the topics
ggplot(
  word_probs,
  aes(term2,beta,fill = as.factor(topic))
) + geom_col(show.legend = FALSE) +
  facet_wrap(~ topic, scales = "free") +
  coord_flip() +
  theme_minimal() +
  theme(panel.grid = element_blank()) +
  labs(x = "Term", y ="Beta")
```

### Bigrams

``` r
# Anonymize messages
text_bigram = removeWords(messages_freeText$message_cleaned, names$word) 
text_bigram = removeWords(text_bigram, stop_names$word) 
text_bigram = removeWords(text_bigram, c("obrigado"))
text_bigram  = Filter(function(x) trimws(x) != "", text_bigram)

# Create bigrams by separating words in sequences of 2
df_corpus <- data.frame(text_bigram)
colnames(df_corpus) = c("text_bigram")

bigrams_df <- df_corpus %>%
  unnest_tokens(output = bigram,
    input = text_bigram,
    token = "ngrams",
    n = 2
  )

bigrams_df %>% count(bigram, sort = TRUE)

bigrams_separated <- bigrams_df %>% separate(bigram, c("word1", "word2"), sep = " ")

# Remove stopwords that may have remained in the corpus
bigrams_filtered <- bigrams_separated %>%
  filter(!word1 %in% stop_words_pt$word) %>%
  filter(!word2 %in% stop_words_pt$word)

# Count the number of times two words are always together
bigram_counts <- bigrams_filtered %>%
  count(word1, word2, sort = TRUE)

# Create network of bigrams
bigram_n = 150
bigram_network <- bigram_counts %>%
  head(bigram_n) %>%
  graph_from_data_frame()

set.seed(2016)

a <- grid::arrow(type = "closed", length = unit(.07, "inches"))

ggraph(bigram_network, layout = "fr") +
  geom_edge_link(aes(edge_alpha = n), show.legend = FALSE,
                 arrow = a, end_cap = circle(.03, 'inches')) +
  geom_node_point(color = "lightblue", size = 2) +
  geom_node_text(aes(label = name), vjust = .7, hjust = 0.3, size=2.5) +
  labs(title="Text bigrams (V5)")
  theme_void()
```
