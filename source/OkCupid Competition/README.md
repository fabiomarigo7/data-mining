# OkCupid Competition 

### Models

* Random forest

### Non-standard R packages

* dplyr
* rpart
* matrixStats




### R code to reproduce the last submission:


```R
#PACCHETTI
library(dplyr)

# READ CSV
train <- read.csv('http://bee-fore.s3-eu-west-1.amazonaws.com/datasets/101.csv', stringsAsFactors = F)
test <- read.csv('http://bee-fore.s3-eu-west-1.amazonaws.com/datasets/102.csv', stringsAsFactors = F)

# TOLGO VARIABILI INUTILI E PREPARO MYDATA (TRAIN E TEST UNITI)
train <- train[,-c(21:28)]
test <- test[,-c(21:28)]
test$Class <- NA
D <- rbind(train,test)

#=================
# 0. PREPROCESSING 
#=================

#------------------------
# 0.A FEATURE ENGINEERING
# offspring aggregata
D$offspring_A <- D$offspring 
D$offspring_A[which(D$offspring == "doesnt_have_kids_and_doesnt_want_any" | D$offspring == "doesnt_want_kids")] <- "doesnt_want"
D$offspring_A[which(D$offspring == "doesnt_have_kids_but_wants_them" | D$offspring == "doesnt_have_kids_but_might_want_them" | D$offspring == "might_want_kids" | D$offspring == "wants_kids" )] <- "wants"
D$offspring_A[which(D$offspring == "has_kids_and_wants_more" | D$offspring == "has_a_kid_and_wants_more" | D$offspring == "has_kids_and_might_want_more" | D$offspring == "has_a_kid_and_might_want_more")] <- "another"
D$offspring_A[which(D$offspring == "has_kids" | D$offspring == "has_kids_but_doesnt_want_more" | D$offspring == "has_a_kid_but_doesnt_want_more" | D$offspring == "has_a_kid")] <- "has_kids"

# pets aggregata
D$pets_A <- D$pets 
D$pets_A[which(grepl("dislikes", D$pets_A))] <- "no_animals"
D$pets_A[which(grepl("has", D$pets_A))] <- "has_animals"
D$pets_A[which(grepl("like", D$pets_A))] <- "likes_animals"

# atheist
D$atheist <- ifelse(D$religion == "atheism", "yes", "no")

# smoke
D$smoke <- ifelse(D$smokes == "yes", "yes", "altro")

# single
D$single <- ifelse(D$status == "single", "yes", "altro")

# male ricodifica
D$male <- ifelse(D$male == 1, "m", "f")

# town aggregata
D$town_A <- D$where_town
D$town_A[which(D$town_A == "palo_alto" | D$town_A == "mountain_view" | D$town_A == "san_mateo" | D$town_A == "redwood_city" | D$town_A == "belmont" | D$town_A == "stanford" | D$town_A == "menlo_park")] <- "SV"
D$town_A[which(D$town_A == "alameda" | D$town_A == "albany" | D$town_A == "pacifica" | D$town_A == "san_bruno" | D$town_A == "sausalito" | D$town_A == "south_san_francisco")] <- "high"
D$town_A[which(D$town_A == "daly_city" | D$town_A == "emeryville" | D$town_A == "berkeley" | D$town_A == "hayward" | D$town_A == "larkspur" | D$town_A == "novato" | D$town_A == "oakland" | D$town_A == "richmond" |D$town_A == "san_carlos" |D$town_A == "san_leandro" |D$town_A == "san_lorenzo"| D$town_A == "vallejo")] <- "medium"
D$town_A[which(D$town_A == "benicia" | D$town_A == "burlingame" | D$town_A == "castro_valley" | D$town_A == "el_cerrito" | D$town_A == "hercules" | D$town_A == "martinez" | D$town_A == "mill_valley"  | D$town_A == "other"  | D$town_A == "pleasant_hill"  | D$town_A == "san_pablo" | D$town_A == "san_rafael" | D$town_A == "walnut_creek" | D$town_A == "corte_madera" | D$town_A == "moraga" | D$town_A == "lafayette" | D$town_A == "millbrae" | D$town_A == "orinda" | D$town_A == "pinole" | D$town_A == "san_anselmo" | D$town_A == "fairfax"| D$town_A == "fremont"| D$town_A == "green_brae"| D$town_A == "half_moon_bay"| D$town_A == "el_sobrante"  )] <- "low"

# silicon valley
D$SV <- ifelse(D$town_A == "SV", "yes", "no")

# religion importance ricodificata
D$rel_imp <- D$religion_modifer
D$rel_imp[which(D$religion_modifer == "religion_mod_missing")] <- "missing"
D$rel_imp[which(D$religion_modifer == "and_laughing_about_it")] <- "0"
D$rel_imp[which(D$religion_modifer == "but_not_too_serious_about_it")] <- "1"
D$rel_imp[which(D$religion_modifer == "and_somewhat_serious_about_it")] <- "2"
D$rel_imp[which(D$religion_modifer == "and_very_serious_about_it")] <- "3"

# sign importance ricodificata
D$sign_imp <- D$sign_modifer
D$sign_imp[which(D$sign_modifer == "sign_mod_missing")] <- "missing"
D$sign_imp[which(D$sign_modifer == "but_it_doesnt_matter")] <- "0"
D$sign_imp[which(D$sign_modifer == "and_its_fun_to_think_about")] <- "1"
D$sign_imp[which(D$sign_modifer == "and_it_matters_a_lot")] <- "2"

# et? aggregata
D$age_A <- D$age
D$age_A[which(D$age_A < 21)] <- "18-21"
D$age_A[which(D$age_A >= 40 & D$age_A <= 44)] <- "40-44"
D$age_A[which(D$age_A >= 45 & D$age_A <= 49)] <- "45-49"
D$age_A[which(D$age_A >= 50)] <- "50+"

D$age_dummy='no'
D$age_dummy[which(D$age_A >= 25 & D$age_A <= 40)] <- "yes"

# body type aggregata
D$body_type_A <- D$body_type
D$body_type_A[which(D$body_type_A == "curvy" | D$body_type_A == "full_figured"
                    | D$body_type_A == "overweight" | D$body_type_A == "used_up")] <- "over_size"
D$body_type_A[which(D$body_type_A == "athletic" | D$body_type_A == "jacked")] <- "athletic"
D$body_type_A[which(D$body_type_A == "body_type_missing" | D$body_type_A == "rather_not_say")] <- "body_type_missing"
D$body_type_A[which(D$body_type_A == "skinny" | D$body_type_A == "thin")] <- "thin"

# diet aggregata
D$diet_A <- D$diet
D$diet_A[which(grepl('vegan', D$diet_A))] <- "vegan"
D$diet_A[which(grepl('vegetarian', D$diet_A))] <- "vegetarian"
D$diet_A[which(grepl('kosher', D$diet_A) | grepl('mostly_halal', D$diet_A) | grepl('other', D$diet_A))] <- "other"
D$diet_A[which(grepl('other', D$diet_A))] <- "other"
D$diet_A[which(grepl('anything', D$diet_A))] <- "anything"

# drinks aggregata
D$drinks_A <- D$drinks
D$drinks_A[which(D$drinks_A == "desperately" | D$drinks_A == "very_often")] <- "very_often"

# education aggregata 
D$education_A <- D$education
D$education_A[which(D$education_A == "college_university" | D$education_A == "graduated_from_college_university")] <- "university"
D$education_A[which(grepl('dropped_out', D$education_A))] <- "dropped"
D$education_A[which(D$education == "dropped_out_of_college_university")] <- "dropped_university"
D$education_A[which(D$education_A == "graduated_from_high_school" | D$education_A == "graduated_from_law_school"
                    | D$education_A == "graduated_from_med_school" | D$education_A == "working_on_two_year_college"
                    | D$education_A == "working_on_college_university" | D$education_A == "two_year_college"
                    | D$education_A == "space_camp" | D$education_A == "law_school" 
                    | D$education_A == "high_school" )] <- "other"
D$education_A[which(D$education_A == "working_on_space_camp" | D$education_A == "working_on_med_school"
                    | D$education_A == "working_on_law_school" | D$education_A == "working_on_high_school"
                    | D$education_A == "graduated_from_two_year_college")] <- "work_other"
D$education_A[which(D$education_A == "graduated_from_masters_program" | D$education_A == "masters_program" | D$education_A == "graduated_from_space_camp")] <- "masters_program"
D$education_A[which(D$education_A == "graduated_from_ph_d_program" | D$education_A == "ph_d_program")] <- "phd_program"
D$education_A[which(D$education_A == "working_on_masters_program" | D$education_A == "working_on_ph_d_program")] <- "work_phd_mast"

# dummy dottorato
D$phdYN <- "no"
D$phdYN[which(D$education_A == "phd_program")] <- "yes"

# dummy education
D$educ_dummy1 <- "yes"
D$educ_dummy1[which(D$education_A!="ed_missing"&D$education_A!="other"&
                      D$education_A!="work_phd"&D$education_A!="work_phd_mast")] <- "no"
D$educ_dummy2 <- "no"
D$educ_dummy2[which(D$education_A=="masters_program"|D$education_A=="university"|
                      D$education_A=="work_other")] <- "yes"

# altezza
D$height_A <- D$height
D$height_A[which(D$height_A <= 61)] <- "61-"
D$height_A[which(D$height_A >= 75)] <- "75+"

# income 100.000 dummy
D$income100000 <- "no"
D$income100000[which(D$income == "inc100000")] <- "yes"

# variabili unioni delle dummy
col.union <- function(df){
  df <- as.data.frame(sapply(df, factor))
  for (i in 1:ncol(df)){levels(df[,i])<-c("0","1")}; rm(i)
  df <- as.data.frame(sapply(df, as.character), stringsAsFactors = F)
  df <- as.data.frame(sapply(df, as.numeric))
  S <- rowSums(df)
  return(ifelse(S==0, 'no', 'yes'))
}

D$UnionDummy <-  col.union(df = D[,c("tech","computer","SV")])

#---------------------
# 0.B VARIABILI FACTOR
# status,male,smokes,class,+tutte le 0/1 finali
get_positions = function(dataset,names){
  v = colnames(dataset)
  v2 = c()
  for (name in names){
    k = which(v==name)
    v2 = c(v2, k)
  }
  return(sort(v2))
}

var1 = get_positions(D,c('male','Class', colnames(D)[24:length(D)]))
D[,var1] = lapply(D[,var1], as.factor)
for(i in 24:83){
  levels(D[,i]) <- c("no", "yes")  
}
D$essay_link = as.factor(D$essay_link)
levels(D[,'essay_link']) <- c("no", "yes")  

#-----------------------------------------------
# 0.C DATASET FINALE
var2=get_positions(D, c('essay_link', 'essay_length'))
var2=sort(c(var1,var2))
#trasformo i character in factor
D <- mutate_if(D, is.character, as.factor)
train = D[1:4000, var2]
test = D[4001:5000, var2]

#---------------
#  FIT METHOD
#---------------

rfb.fit = function(X_train, y_train, fixed_attrs=0, theta=1, B=1000, alpha=1/3, max_features=0, small_class=NA, reins=T,maxdepth=30,minsplit=20,minbucket=7,seme=1:B){
  library(rpart)
  #library(tree)
  
  if (is.na(small_class)){
    C1 = levels(y_train)[which.min(c(length(y_train[y_train==levels(y_train)[1]]),
                                    length(y_train[y_train==levels(y_train)[2]])))]
    Class_1 = which(y_train == C1)
    Class_0 = which(y_train != C1)  
  }  else {C1 = small_class}
  
  C0 <- levels(y_train)[levels(y_train)!=C1]
  Class_1 = which(y_train == C1)
  Class_0 = which(y_train == C0) 
  
  size = as.integer(length(Class_1)*theta)
  p = ncol(X_train)
  
  trees <- list()
  cols <- list()
  
  for (b in 1:B){
    set.seed(seme[b])
    
    # estrazione righe
    i_1 = sample(Class_1, size+10, replace = reins)
    i_0 = sample(Class_0, size-10, replace = reins)
    i = sort(c(i_0,i_1))
    
    # estrazione colonne
    if (max_features<=1) {j = sort(sample(1:p, size = as.integer(p*alpha), replace = FALSE))
    } else {j = sort(sample(1:p, size = max_features, replace = FALSE))}
    
    if(length(fixed_attrs)>1){
      fixed = get_positions(X_train,fixed_attrs)
      j = union(j,fixed)
    }
    cols[[b]] <- colnames(X_train)[j]
    
    # campione bootstrap
    help_df = data.frame(X_train[i,j], CLASS = y_train[i])
    
    # modello albero (rpart)    

    ctrl = rpart.control(maxdepth=maxdepth,minbucket=minbucket, minsplit=minsplit,xval=1)
    trees[[b]] <- rpart(CLASS~.,data=help_df,control=ctrl)
    
  }
  output <- list(Trees=trees, Cols=cols, Class1=C1, Class0=C0)
  return(output)
}

#-----------------
# PREDICT METHOD
#-----------------

rfb.predict <- function(rfb_object, newdata, method='class'){
  
  library(matrixStats)
  B = length(rfb_object$Trees)
  
  # METHOD = CLASS
  if (method=='class'){
    matriX=c()
    for (b in 1:B){
      j = rfb_object$Cols[[b]]
      fit = rfb_object$Trees[[b]]
      p = as.character(predict(fit, newdata=newdata[,j], type='class'))
      matriX = cbind(matriX,p)}
    fitted = c()
    for (k in 1:nrow(matriX)){
      proP = sum(matriX[k,] == rfb_object$Class1)/ncol(matriX)
      if(proP>0.5) {fitted[k] = rfb_object$Class1}
      else if(proP<0.5) {fitted[k]=rfb_object$Class0}
      else if(proP==0.5) {fitted[k]=sample(c(rfb_object$Class0,rfb_object$Class1), size=1)}}
    return(as.factor(fitted))
  }
  
  # METHOD = PROPORTION
  else if (method=='proportion'){
    matriX=c()
    for (b in 1:B){
      j = rfb_object$Cols[[b]]
      fit = rfb_object$Trees[[b]]
      p = as.character(predict(fit, newdata=newdata[,j], type='class'))
      matriX = cbind(matriX,p)}
    fitted = c()
    for (k in 1:nrow(matriX)){
      fitted[k] = sum(matriX[k,] == rfb_object$Class1)/ncol(matriX)}
    return(fitted)
  }
  
  # METHOD = MEAN
  else if (method=='mean'){
    matriX=c()
    for (b in 1:B){
      j = rfb_object$Cols[[b]]
      fit = rfb_object$Trees[[b]]
      p = predict(fit, newdata=newdata[,j])[,rfb_object$Class1]
      matriX = cbind(matriX,p)}
    fitted = rowMeans(matriX)
    return(fitted)
  }
  
  # METHOD = MEDIAN
  else if (method=='median'){
    matriX=c()
    for (b in 1:B){
      j = rfb_object$Cols[[b]]
      fit = rfb_object$Trees[[b]]
      p = predict(fit, newdata=newdata[,j])[,rfb_object$Class1]
      matriX = cbind(matriX,p)}
    fitted = rowMedians(matriX)
    return(fitted)
  }
}

#=============================
# 1. SUBMISSION MODEL
#=============================

# VARIABILI GLM
attrs <- c("male", "essay_link", "tech", "computer", "science", "fixing", 
           "matrix", "electronic", "nerdy", "artist", "SV", "body_type_A", "diet_A", 
           "single", "student", "teaching", "loyal", "atheist", "smoke", "sign_imp", 
           "education_A", "income100000", "phdYN", "Class") 

train = train[,attrs]; test = test[,attrs]

# PREPARO TRAIN E TEST
X_test <- test; X_test$Class=NULL
X_train <- train; X_train$Class=NULL
y_train <- train$Class

#RANDOM FOREST
pred.tot <- c()
B <- 5000
h.mat <- c()
#RANDOM FOREST
pred.tot <- c()
B <- 5000
h.mat <- c()
for(g in 1:100){
  set.seed(g)
  h <- sample(1:100000, B,  replace=F)
  h.mat <- cbind(h.mat,h)
}
for(i in 1:100){
  fit.tot <- rfb.fit(X_train, y_train, theta=1, alpha=0.15, B=B, 
                     minsplit=10, minbucket=10, seme=h.mat[, i])
  pred.tot <- cbind(pred.tot, rfb.predict(fit.tot, newdata = X_test, method="median"))
}
y_pred_finale <- apply(X = pred.tot, MARGIN = 1 , FUN = mean, trim = .2)

# SUBMISSION
write.table(file="./submission/OkCupid_Submission_finale.txt", y_pred_finale, row.names = FALSE, col.names = FALSE)

head(y_pred_finale)
```

```
[1] 0.5313592 0.5312175 0.4751308 0.4541245 0.4763871 0.4811057
```


