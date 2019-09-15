---
title: "inspecting_factors"
output: html_notebook
---

This saves the loadings to a dataFrame and renames the columns
```{r}
#this saves the factor loadings to a data frame
fa_loadings <- fac$loadings%>%
  as.table()%>%
  as.data.frame()%>%
  filter(Freq>=0.3 | Freq<=-0.3)%>%
  arrange(desc(Var2))

colnames(fa_loadings)<-c("Question", "Factor", "Loading")
```

This sets up a list that allows you to find the text of each question using questions$Q#
```{r}
#set up empty list:
questions<-vector(mode = "list", length =26)

#set up an empty list of names:
qnames<-vector(mode = "character", length = 26)

#read in the the part of the codebook that lists the questions to a temporary file
temp<-readLines("NIS_data/codebook.txt")[9:34]

#the first item of each line in temp is the question # and the 2nd item is the question text:
for (i in 1:26){
  qnames[i]<-strsplit(temp[i], '\t')[[1]][1]
  questions[i]<-strsplit(temp[i], '\t')[[1]][2]
  names(questions)<-qnames
}

#clean up:
rm(i,temp,qnames)

#Example
questions$Q18
```

This adds a column to the fa_loadings dataFrame that is the text of each question
```{r}
fa_loadings$Question_txt<-factor(NA,levels = c("I use my hands and arms to gesture while talking to people. ","I touch others on the shoulder or arm while talking to them. ","I use a monotone or dull voice while talking to people. ","I look over or away from others while talking to them. ","I move away from others when they touch me while we are talking. ","I have a relaxed body position when I talk to people. ", "I frown while talking to people. ","I avoid eye contact while talking to people. ", "I have a tense body position while talking to people. ","I sit close or stand close to people while talking with them. ","My voice is monotonous or dull when I talk to people. ","I use a variety of vocal expressions when I talk to people. ", "I gesture when I talk to people. ","I am animated when I talk to people. ","I have a bland facial expression when I talk to people. ","I move closer to people when I talk to them. ","I look directly at people while talking to them. ","I am stiff when I talk to people. ","I have a lot of vocal variety when I talk to people. ","I avoid gesturing while I am talking to people. ","I lean toward people when I talk to them. ","I maintain eye contact with people when I talk to them. ","I try not to sit or stand close to people when I talk with them. ","I lean away from people when I talk to them. ","I smile when I talk to people. ","I avoid touching people when I talk to them."))

for (i in unique(fa_loadings$Question)){
  fa_loadings$Question_txt[fa_loadings$Question==i]<-as.character(questions[i])
}
```


This makes a semPlot object from the loadings, and create a visualization of the Questions and their factor loadings above 0.3
```{r}
model<-semPlotModel(fac$loadings)
semPaths(model,what="par",whatLabels="est", rotation=4, minimum=0.3, sizeMan=5, sizeMan2 = 2, edge.width =.3)
```


This compiles a list of all the questions and loadings in each factor, as lists, and prints each one for inspection.
```{r}
factors<-vector(mode = "list", length =8)
fnames<-vector(mode = "character", length = 8)

for (i in 1:length(unique(fa_loadings$Factor))){
  factors[[i]]<-assign(paste("PA", i, sep=''),fa_loadings%>%
    filter(Factor==paste("PA",i, sep=""))%>%
    filter(Loading >= 0.3 | Loading <= -0.3)%>%
    select(c(Loading,Question,Question_txt)))
  fnames[i]<-as.character(unique(fa_loadings$Factor))[i]
  names(factors)<-sort(fnames)
  print(factors[[i]])
}
```
