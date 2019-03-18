# Regression-Analysis-and-fitting-of-distribution-on-Storj-token-dataset-in-R
## Abstract 
The aim of the project is to implement different statistical methods on the Storj Token transaction data.The results of the methods are analyzed to derive inferences about the token data.In 4.1, we found how many times a user buys or sells the STORJ token, then fit a distribution and estimate the best distribution.In 4.2, we find the correlation between the STORJ Token price data and layer features of the STORJ Token.In 4.3, we find the most active buyer and seller of our STORJ token and then track them in all other tokens, plot how many unique tokens have they invested in the provided time frame then fit and estimate distributions as part 1. In 4.4, we created  a multiple linear regression model to explain price return on day t. 

**source code:** https://github.com/vineetamonkar/r_project_2/blob/master/StatFinalReport.Rmd


## 1. Introduction:

### 1.1 Important Concepts

Before we get into the project details it is important to note few concepts.
Below we summarise these terms:

#### 1.1.1 Blockchain:
It is a public ledger formed of multiple blocks.Each block contains crytographic hash of the previous block, transaction data, timestamp and other metadata.The blockchain is a chain of these blocks linked together cryptographically and stored on a distributed peer-to-peer network.

#### 1.1.2 Ethereum:
Ethereum is a blockchain platform for digital currency and for builiding decentralized applications using smart contracts.
Smart Contract is a set of instructions of the format if-this-then-that, it is formed when someone needs to performing particular task involving one or more than one entities of the blockchain. The contract is a code which executes itself on occurence of a triggering event such as expiration date.The smart contracts can be written with different languages such as solidity 
The EVM is a runtime environment for smart contracts in Ethereum. Every Ethereum node in the network runs an EVM implementation and executes the same instructions.
Ether is the digital currency(cryptocurrency) of Ethereum. Every individual transaction or step in a contract requires some computation. To perform any computation user has to pay a cost calulated in terms of 'Gas' and paid in terms of 'Ether'. The Gas consist of two parts:

Gas limit: It is the amount of units of gas
Gas price: It is the amount paid per unit of gas

#### 1.1.3 Ethereum ERC-20 Tokens:
If a user needs some service provided by the DAPPS, then he has to pay for that service in terms of 'token' associated with the DAPPS. These Ethereum tokens can be bought using Ether or other cryptocurrencies and can serve the following two purposes:

1) Usage Token: These tokens are used to pay for the services of the Dapp
2) Work Token: These tokens identify you as a shareholder in the DAPP

ERC-20 is a technical standard used for smart contracts on the Ethereum Blockchain for implementing tokens. It is a common list of rules for Ethereum token regarding interactions between tokens,transferring tokens between addresses and how data within th e token is accessed

### 1.2 Project Primary Token: Storj

#### 1.2.1 Storj:

Storj is a decentralized cloud storage network. It uses a blockchain based hash table to store files on a peer-to-peer network.
It consists of two entities farmers and tenants:

Tenants: They are the users who upload their data to cloud storage.
Farmers: They are the different nodes in the network who host the data.

Characteristics of Storj:  
1) Sharding : data is split into different parts and no one farmer has the entire data stored on his node.  
2) End-to-End encryption: The files are encrypted before uploading.  
3)File Verification: Regular Audits are performed to verify the files  
4)Redundancy  

Distributed Hash table (DHT):
It consist of key-value pairs. The keyspace is split among the participating nodes. One method of DHT implementation: The name of the file is hashed to a key. The ownership of this key lies with one of the nodes. The network is traversed for that node which then stores that data and key. While retrieval the filename is hashed again to give the same key and that node is searched again which provides the corresponding value.

#### 1.2.2 Storj Token:

STORJ tokens are utilized as the payment method on the STORJ network.
Details (11/2/2018):  

1 STORJ = $0.334425 USD  
Market Cap = $45,114,748 USD  
Circulating Supply = 135,787,439 STORJ  
Total Supply = 424,999,998 STORJ  
Subunits =  $10^8$   

## 2. Data Description

The Data used for the project is divided in two parts:

1) Token network edge files:

There are 40 Token network edge files.Token edge files have 4 columns: from_node, to_node, unix-time, total-amount.For each row it implies that from_node sold total-amount of the token to to_nodeid at time unix-time.

 (a) from_node : Id which sells the token in the transaction  
 (b) to_node : Id which buys the token in the transaction  
 (c) unixtime : Unix time of the transaction  
 (d) totalamount : Total amount of the storj token involved in the transaction
For Part 1,2 and 4 of the project we will only use the STORJ token network edge file. For part 3 we will use the token network edge files of all 40 tokens.

2) Token price files:
Price dataset for storj token it contains 379 rows and 7 columns as follows:  
 (a) Date  
 (b) Open : Opening price of the token on that day  
 (c) High : Max price of the token on that day  
 (d) Low : Min price of the token on that day  
 (e) Close : Closing price of the token on that day  
 (f) Volume : Volume of the token on that day  
 (g) Market Cap: Market Cap of that token on that day
 We use price data in part 2 and 4.

## 3. Preprocessing

There could be some records in the transactions where total amount is very large. Some of this records could be due to some bug or glitch(integer overflow problem). These values can be separated from data using a threshold value.

Calculating this value for the STORJ token:The value of transaction amount can't be greater than the max value where, max value = total supply of tokens * subunits.subsituting the values from above.

```{r}
# Load Data
tokenData <- read.delim("networkstorjTX.txt", header = FALSE, sep = " ")
tokenFrame<- as.data.frame(tokenData)
colnames(tokenFrame) <- c("fromNode","toNode","Date","totalAmount")
# Find Outliers:
TotalSupply <- 424999998
Decimals <- 10^8
OutlierValue <- TotalSupply*Decimals
Outlierdata <- tokenFrame[ which(tokenFrame$totalAmount > OutlierValue),]
# Total Number Of Outliers:
message("Total number of outliers: ", length(Outlierdata$totalAmount))
# How Many Users Are Included In Outliers Transactions:
users <- c(Outlierdata$fromNode,Outlierdata$toNode)
uniqueUsers<- unique(users)
message(length(uniqueUsers), " users are includednin outliers transactions")
# Remove Outliers
WithoutOutlierdata <- tokenFrame[ which(tokenFrame$totalAmount < OutlierValue),]
```

## 4.Token Data Analysis 

### 4.1 Finad and Fit Distribution

**The package we use to fit distribution: ** fitdistrplus, provide the function fitdist() we can use to fit distribution of our data.
**The function we use to fit distribution: ** fitdist(). Fit of univariate distributions to different type of data with different estimate method we can choose: maximum likelihood estimation (mle), moment matching estimation (mme), quantile matching estimation (qme), maximizing goodness-of-fit estimation (mge), the default and we mostly used one is MLE.
Output of this function is S3 object, we can use several methods like plot(), print(), summary() to visualize it or get more detailed information.We use plot in this project.
```{r}
frequencyTable <- table(WithoutOutlierdata[2])
freq<- as.data.frame(frequencyTable)
colnames(freq) <- c("buyerID","frequency")
FrequencyBuyers = table(freq$frequency)
freqNoBuys = as.data.frame(FrequencyBuyers)
colnames(freqNoBuys) <- c("NoBuys","freqNoBuys")
# barplot(freqNoBuys$freqNoBuys,names.arg = freqNoBuys$NoBuys,ylab = "Frequency of Number of Buyers", xlab = "Number of Buyers", xlim = c(0,20),ylim = c(0,140000))
```
```{r fig.cap = "Fitting with Weibull Distribution-Buys"}
# fit distribution
# Poisson Distribution
library(fitdistrplus)
fit <- fitdist(freqNoBuys$freqNoBuys, "pois", method="mle")
# Weibull Distribution
fit2 <- fitdist(freqNoBuys$freqNoBuys, "weibull",method = "mle")
# Exponential Distribution
fit3 <- fitdist(freqNoBuys$freqNoBuys, "exp",method = "mme")
# Geometric Distribution
fit4 <- fitdist(freqNoBuys$freqNoBuys, "geom",method = "mme")
# Normal Distribution
fit5 <- fitdist(freqNoBuys$freqNoBuys, "norm")
plot(fit2)
```
The sells part is similiar to buys part, so we didn't show the code in this report, the follow is the plot of sells:
```{r echo=FALSE}
# Fit Distribution -- Sells
frequencyTable <- table(WithoutOutlierdata[1])
freq<- as.data.frame(frequencyTable)
colnames(freq) <- c("sellerID","frequency")
FrequencySellers = table(freq$frequency)
freqNoSells = as.data.frame(FrequencySellers)
colnames(freqNoSells) <- c("NoSells","freqNoSells")
# barplot(freqNoSells$freqNoSells,names.arg = freqNoSells$NoSells,ylab = "Frequency of Number of Sellers", xlab = "Number of Sellers", xlim = c(0,20),ylim = c(0,50000))
```
```{r echo=FALSE, fig.cap = "Fitting with Weibull Distribution-Selles"}
# fit distribution
# Poisson Distribution
library(fitdistrplus)
fitSells <- fitdist(freqNoSells$freqNoSells, "pois", method="mle")
# Weibull Distribution
fitSells2 <- fitdist(freqNoSells$freqNoSells, "weibull",method = "mle")
# Exponential Distribution
fitSells3 <- fitdist(freqNoSells$freqNoSells, "exp",method = "mme")
# Geometric Distribution
fitSells4 <- fitdist(freqNoSells$freqNoSells, "geom",method = "mme")
# Normal Distribution
fitSells5 <- fitdist(freqNoSells$freqNoSells, "norm")
plot(fitSells2)
```
### 4.2 Calulate Correlations of Token Data and Price Data
```{r}
# Change the Date Format
WithoutOutlierdata$Date <- as.Date(as.POSIXct(WithoutOutlierdata$Date,origin="1970-01-01",tz="GMT"))
# Load Price Date and Change the Date Format
priceData <- read.delim("storj", header = TRUE, sep = "\t")
priceData$Date <- as.Date(priceData$Date,"%m/%d/%Y")
# Join Two Data by Date
CombineData <- merge(x = WithoutOutlierdata, y = priceData, by = "Date")
buyerstable <- aggregate(data=CombineData, toNode ~ Date, function(x) length(unique(x)))
colnames(buyerstable) <- c("Date","unique_buyers")
CombineData <- merge(CombineData,buyerstable,by = "Date")
CombineData <- unique(CombineData[,c(1,4:11)])
## Create layers and calulate correlations
library(dplyr)
alpha <- c(1e-14, 1e-12, 1e-10, 1e-9, 4e-9, 5e-9, 6e-9, 8e-9, 1e-8, 1e-7, 1e-6, 1e-5, 1e-4, 1e-3, 1e-2)
maxValue <- max(WithoutOutlierdata$totalAmount)
for(a in alpha){
  value <- a*maxValue
  layer <- CombineData[ which(CombineData$totalAmount < value),]
  correlation <- cor(layer$unique_buyers, layer$Open,use="complete.obs",method = "pearson")
  print(correlation)
}
```
### 4.3 Most Active Buyer and Seller Analysis
```{r}
# Find most active buyer in our token:
frequencyTable <- table(WithoutOutlierdata[2])
freq<- as.data.frame(frequencyTable)
colnames(freq) <- c("buyerID","frequency")
mostFreq <- freq[which(freq$frequency== max(freq$frequency)),]
buyer <- mostFreq$buyerID
message("Id of the most active buyer in our token: ", buyer)
```
```{r}
# Plot the number of transactions of each month of buyer5:
user5 <- WithoutOutlierdata[which(WithoutOutlierdata$toNode == buyer),]
user5$Date <- as.Date(as.POSIXct(user5$Date,origin="1970-01-01",tz="GMT"))
user5$Date <- cut(user5$Date, breaks = "month")
frequencyTable5 <- table(user5[3])
freq5 <-  as.data.frame(frequencyTable5)
colnames(freq5) <- c("time","frequency")
# barplot(freq5$frequency,names.arg = freq5$time,ylab = "number of transactions", xlab = "month")
```
```{r}
# Load all token data
filenames <- list.files("tokens", pattern="*.txt", full.names=TRUE)
#find the minimum and max date of all token data
newfreq5 <- freq5[order(freq5$time),]
minDate <- as.Date(newfreq5$time[1])
maxDate <- as.Date(newfreq5$time[nrow(newfreq5)])
for (file in filenames){
  token <- read.delim(file, header = FALSE, sep = " ")
  Frame<- as.data.frame(token)
  colnames(Frame)<-c('fromNode','toNode','unixTime','totalAmount')
  user5_other <- Frame[which(Frame$toNode == buyer),]
  if (nrow(user5_other)>0){
    user5_other$unixTime <- as.Date(as.POSIXct(user5_other$unixTime,origin="1970-01-01",tz="GMT"))
    user5_other$unixTime <- cut(user5_other$unixTime, breaks = "month") 
    frequencyTable5_other <- table(user5_other[3])
    freq5_other <-  as.data.frame(frequencyTable5_other)
    colnames(freq5_other) <- c("time","frequency")
    newfreq5_other <- freq5_other[order(freq5_other$time),]
    tempminDate <- as.Date(newfreq5_other$time[1])
    tempmaxDate <- as.Date(newfreq5_other$time[nrow(newfreq5_other)])
    if (tempminDate<minDate){
      minDate <- tempminDate
    }
    if (tempmaxDate>maxDate){
      maxDate <- tempmaxDate
    }
  }
}
# Base on the minmum and maxmum date, create an array of counter and date intervals
countArray <- c(0,0,0,0,0,0,0,0,0,0)
dateinterval <- c("2017-08-01","2017-09-01","2017-10-01","2017-11-01","2017-12-01","2018-01-01","2018-02-01","2018-03-01","2018-04-01","2018-05-01")
dateinterval <- as.Date(dateinterval)
# Count the number of unique tokens that buyer5 bought each month
for (file in filenames){
  token2 <- read.delim(file, header = FALSE, sep = " ")
  Frame2<- as.data.frame(token2)
  colnames(Frame2)<-c('fromNode','toNode','unixTime','totalAmount')
  user5_other2 <- Frame2[which(Frame2$toNode == buyer),]
  
  if (nrow(user5_other2)>0){
    user5_other2$unixTime <- as.Date(as.POSIXct(user5_other2$unixTime,origin="1970-01-01",tz="GMT"))
    user5_other2$unixTime <- cut(user5_other2$unixTime, breaks = "month") 
    frequencyTable5_other2 <- table(user5_other2[3])
    freq5_other2 <-  as.data.frame(frequencyTable5_other2)
    colnames(freq5_other2) <- c("time","frequency")
    freq5_other2$time <- as.Date(freq5_other2$time)
    for (arow in 1:nrow(freq5_other2))
    {
      if (freq5_other2$time[arow]==dateinterval[1]){
        countArray[1] = countArray[1]+1
        
      }
      else if (freq5_other2$time[arow]==dateinterval[2]){
        countArray[2] = countArray[2]+1
      }
      else if (freq5_other2$time[arow]==dateinterval[3]){
        countArray[3] = countArray[3]+1
      }
      else if (freq5_other2$time[arow]==dateinterval[4]){
        countArray[4] = countArray[4]+1
      }
      else if (freq5_other2$time[arow]==dateinterval[5]){
        countArray[5] = countArray[5]+1
      }
      else if (freq5_other2$time[arow]==dateinterval[6]){
        countArray[6] = countArray[6]+1
      }
      else if (freq5_other2$time[arow]==dateinterval[7]){
        countArray[7] = countArray[7]+1
      }
      else if (freq5_other2$time[arow]==dateinterval[8]){
        countArray[8] = countArray[8]+1
      }
      else if (freq5_other2$time[arow]==dateinterval[9]){
        countArray[9] = countArray[9]+1
      }
      else if (freq5_other2$time[arow]==dateinterval[10]){
        countArray[10] = countArray[10]+1
      }
      else {
        print("error")
      }
    }
  }
}
# plot
dateinterval2 <- c("Aug-17","Sep-17","Oct-17","Nov-17","Dec-17","Jan-18","Feb-18","Mar-18","Apr-18","May-18")
# barplot(countArray,names.arg = dateinterval2,ylab = "no of unique tokens", xlab = "month")
```
```{r fig.cap = "Fitting with Normal Distribution - Most Active Buyer"}
# fir distribution
# 1.Exponential Distribution
library(fitdistrplus)
fit <- fitdist(countArray, "exp", method="mle")
# ks.test(countArray,"pexp",0.05,alternative = c("two.sided", "less", "greater"), exact = NULL)
# 2.Weibull Distribution
fit2 <- fitdist(countArray, "weibull",method = "mle")
# ks.test(countArray,"pweibull",3.938559,alternative = c("two.sided", "less", "greater"), exact = NULL)
# 3.Poisson Distribution
fit3 <- fitdist(countArray, "pois", method="mle")
# ks.test(countArray,"ppois",19.9,alternative = c("two.sided", "less", "greater"), exact = NULL)
# 4.Geometric Distribution
fit4 <- fitdist(countArray, "geom",method = "mme")
# ks.test(countArray,"pgeom",0.04784689,alternative = c("two.sided", "less", "greater"), exact = NULL)
# 5. Normal Distribution
fit5 <- fitdist(countArray, "norm")
ks.test(countArray,"pnorm",19.9,6.3,alternative = c("two.sided", "less", "greater"), exact = NULL)
plot(fit5)
```
The sellers part is similiar to the above, we didn't show the code in this report.
```{r echo=FALSE}
# Seller part
# Find most active seller in our token
frequencyTableSeller <- table(WithoutOutlierdata[1])
freqseller<- as.data.frame(frequencyTableSeller)
colnames(freqseller) <- c("sllerID","frequency")
mostFreqSeller <-freqseller[which(freqseller$frequency == max(freqseller$frequency)),]
seller <- mostFreqSeller$sllerID
# message("Id of the most active seller in our token: ", seller)
```
```{r echo=FALSE}
# Plot the number of transactions of each month of seller6253227(we call it user2)
user2 <- WithoutOutlierdata[which(WithoutOutlierdata$fromNode == seller),]
user2$Date <- as.Date(as.POSIXct(user2$Date,origin="1970-01-01",tz="GMT"))
user2$Date <- cut(user2$Date, breaks = "month")
frequencyTable2 <- table(user2[3])
freq2 <-  as.data.frame(frequencyTable2)
colnames(freq2) <- c("time","frequency")
# barplot(freq2$frequency,names.arg = freq2$time,ylab = "number of transactions", xlab = "month",ylim = c(0,70000))
```
```{r echo=FALSE}
# Read all token data
filenames <- list.files("tokens", pattern="*.txt", full.names=TRUE)
# find the minimum and max date of all token data
# initial minDate and maxDate
newfreq2 <- freq2[order(freq2$time),]
minDate <- as.Date(newfreq2$time[1])
maxDate <- as.Date(newfreq2$time[nrow(newfreq2)])
                   
for (file in filenames){
  token <- read.delim(file, header = FALSE, sep = " ")
  Frame<- as.data.frame(token)
  colnames(Frame)<-c('fromNode','toNode','unixTime','totalAmount')
  user2_other <- Frame[which(Frame$fromNode == seller),]
  if (nrow(user2_other)>0){
    user2_other$unixTime <- as.Date(as.POSIXct(user2_other$unixTime,origin="1970-01-01",tz="GMT"))
    user2_other$unixTime <- cut(user2_other$unixTime, breaks = "month") 
    frequencyTable2_other <- table(user2_other[3])
    freq2_other <-  as.data.frame(frequencyTable2_other)
    colnames(freq2_other) <- c("time","frequency")
    newfreq2_other <- freq2_other[order(freq2_other$time),]
    tempminDate2 <- as.Date(newfreq2_other$time[1])
    tempmaxDate2 <- as.Date(newfreq2_other$time[nrow(newfreq2_other)])
    if (tempminDate<minDate){
      minDate <- tempminDate
    }
    if (tempmaxDate>maxDate){
      maxDate <- tempmaxDate
    }
  }
}
```
```{r echo=FALSE}
# Base on the minmum and maxmum date, create an array of counter and date intervals
countArraySeller <- c(0,0,0,0,0,0,0,0,0,0,0)
dateinterval <- c("2017-07-01","2017-08-01","2017-09-01","2017-10-01","2017-11-01","2017-12-01","2018-01-01","2018-02-01","2018-03-01","2018-04-01","2018-05-01")
dateinterval <- as.Date(dateinterval)
# Count the number of unique tokens that the most active seller bought each month
for (file in filenames){
  token2 <- read.delim(file, header = FALSE, sep = " ")
  Frame2<- as.data.frame(token2)
  colnames(Frame2)<-c('fromNode','toNode','unixTime','totalAmount')
  user2_other2 <- Frame2[which(Frame2$fromNode == seller),]
  
  if (nrow(user2_other2)>0){
    user2_other2$unixTime <- as.Date(as.POSIXct(user2_other2$unixTime,origin="1970-01-01",tz="GMT"))
    user2_other2$unixTime <- cut(user2_other2$unixTime, breaks = "month") 
    frequencyTable2_other2 <- table(user2_other2[3])
    freq2_other2 <-  as.data.frame(frequencyTable2_other2)
    colnames(freq2_other2) <- c("time","frequency")
    freq2_other2$time <- as.Date(freq2_other2$time)
    for (arow in 1:nrow(freq2_other2))
    {
      if (freq2_other2$time[arow]==dateinterval[1]){
        countArraySeller[1] = countArraySeller[1]+1
        
      }
      else if (freq2_other2$time[arow]==dateinterval[2]){
        countArraySeller[2] = countArraySeller[2]+1
      }
      else if (freq2_other2$time[arow]==dateinterval[3]){
        countArraySeller[3] = countArraySeller[3]+1
      }
      else if (freq2_other2$time[arow]==dateinterval[4]){
        countArraySeller[4] = countArraySeller[4]+1
      }
      else if (freq2_other2$time[arow]==dateinterval[5]){
        countArraySeller[5] = countArraySeller[5]+1
      }
      else if (freq2_other2$time[arow]==dateinterval[6]){
        countArraySeller[6] = countArraySeller[6]+1
      }
      else if (freq2_other2$time[arow]==dateinterval[7]){
        countArraySeller[7] = countArraySeller[7]+1
      }
      else if (freq2_other2$time[arow]==dateinterval[8]){
        countArraySeller[8] = countArraySeller[8]+1
      }
      else if (freq2_other2$time[arow]==dateinterval[9]){
        countArraySeller[9] = countArraySeller[9]+1
      }
      else if (freq2_other2$time[arow]==dateinterval[10]){
        countArraySeller[10] = countArraySeller[10]+1
      }
       else if (freq2_other2$time[arow]==dateinterval[11]){
        countArraySeller[11] = countArraySeller[11]+1
      }
      else {
        print("error")
      }
    }
  }
}
# plot
# dateintervalSeller2 <- c("Jul-17","Aug-17","Sep-17","Oct-17","Nov-17","Dec-17","Jan-18","Feb-18","Mar-18","Apr-18","May-18")
# barplot(countArraySeller,names.arg = dateintervalSeller2,ylab = "no of unique tokens", xlab = "month")
```
### 4.4 Create Linear Regression Model
In this part we created a multiple linear regression model on the price data and our primary token data.Our response variable(y) is simple price return given by $p_{t} -p_{t-1} / p_{t-1}$ where, $p_{t}$ is the token price in dollar for $t_{th}$ day.
The regressor variables(x1...xn) are as follows:
1)No of token transactions per day
2)No of total token bought or sold per day
3)No of unique buyers per day
4)Percentage of investors who bought more than token per day.
```{r echo=FALSE}
tokenData <- read.delim("networkstorjTX.txt", header = FALSE, sep = " ")
tokenFrame<- as.data.frame(tokenData)
colnames(tokenFrame)<-c('fromNode','toNode','unixTime','totalAmount')
priceData <- read.delim("storj", header = TRUE, sep = "\t")
TotalSupply <- 424999998
Decimals <- 10^8
OutlierValue <- TotalSupply*Decimals
WithoutOutlierdata <- tokenFrame[ which(tokenFrame$totalAmount < OutlierValue),]
newwithoutoutlierdata <- WithoutOutlierdata
newwithoutoutlierdata$unixTime <- as.Date(as.POSIXct(WithoutOutlierdata$unixTime,origin="1970-01-01",tz="GMT"))
priceData$Date <- as.Date(priceData$Date, "%m/%d/%Y")
```
Calculate response variable from the price data using open price. The first few records of the response variable as follows:
```{r}
return <- numeric(nrow(priceData))
return[379] <- priceData$Open[379]
for (i in 378:1) {
  return[i] <- (priceData$Open[i]-priceData$Open[i+1])/priceData$Open[i+1]
}
priceData$return <- return
head(priceData$return)
```
Merge both the datasets according to date which would help us in calculating the regressor variables.
```{r}
colnames(newwithoutoutlierdata)<-c('fromNode','toNode','Date','totalAmount')
newdataset <- merge(newwithoutoutlierdata,priceData,by = "Date")
```
```{r echo = FALSE,eval = FALSE}
#token data dates
summary(newwithoutoutlierdata$Date)
#price data dates
summary(priceData$Date)
#dates after merging
summary(newdataset$Date)
```
#### 4.4.1 Model 1:Simple Linear Regression
```{r echo=FALSE}
# Feature Engineering
frequencytrans <- table(newdataset$Date)
freqtrans<- as.data.frame(frequencytrans)
colnames(freqtrans) <- c("Date","no_of_trans")
freqtrans$Date <- as.Date(freqtrans$Date)
head(freqtrans$no_of_trans)
```
```{r echo = FALSE}
newsimpletable <- merge(priceData,freqtrans,by = "Date")
newsimpletable<- newsimpletable[,c(-6:-7)]
```
Data preparation: univariate and bivariate analysis.
**Univariate Analysis**:

Check for outliers and then remove them by capping our data to the max value.To remove majority of the outlier we are capping the data 92% quantile. Then we will alsocheck the response variable. 

```{r echo =FALSE, fig.cap = "Boxplots of variables for Univariate analysis in simple regression"}
par(mfrow = c(1,3))
bx = boxplot(newsimpletable$no_of_trans,ylim = c(0,5000),main = "with outlier",ylab ="no of trans" )
newsimpletable$no_of_trans<-ifelse(newsimpletable$no_of_trans>2275,2274.36,newsimpletable$no_of_trans)
boxplot(newsimpletable$no_of_trans, main ="without outlier",ylab = "no of trans")
#quantile(newsimpletable$no_of_trans, seq(0,1,0.02))
#bx$stats
boxplot(newsimpletable$return, main = "with outliers",ylab ="simple return")
```

But for simple return response varaible case we won't be removing any outliers as we don't want to have any impact on the response variable.

**Bivariate Analysis**:
Draw the scatterplot between the regressor and the response variable.Before we do that it is important to shift our response variable up by one row because we will be using yesterday to model today's response.

```{r }
shift <- function(x, n){
  c(x[-(seq(n))], rep(NA, n))
}
newsimpletable$return <- shift(newsimpletable$return, 1)
newsimpletable <- newsimpletable[1:(nrow(newsimpletable)-1),]
head(newsimpletable$return)
#plot(newsimpletable$no_of_trans,newsimpletable$return)
```

We can see that there isn't much relation between no of transactions each day and the simple return variable. We can confirm this by finding the correlation between the two

```{r}
cor(newsimpletable[,c(6,7)])
```
After completing the analysis we move ahead to fit the linear model.

```{r}
final_data <- lm(return~no_of_trans,data=newsimpletable)
summary(final_data)
```

From the summary of the linear model we can clearly see that it is not a good model, the p-values of the intercept and the regressor are very high, also the r-square conveys only 2% of the response variable.

**Testing**  

We can test this linear model using the plot function:

```{r echo=FALSE ,warning=FALSE, error=FALSE, message=FALSE,fig.cap = "Residual Plot of the linear model"}
library(lmtest)
par(mfrow = c(2,2))
plot(final_data)
```

1. **Residuals vs fitted**:

Even though we see a horizontal line in this plot we can see that the residuals are not evenly spread across the line.It may not suggest a pattern but we can see most of the points in the left speculating a non-linear behaviour

2. **Normal Q-Q**:

The plot leads us to believe that the residuals follow normality as most of the points are seen to lie on the line or near it although few observations such as 42,2 and 156 are significantly away from the line.

3. **Scale-Location**:

The horizontal line in this plot suggests that homoscedasticity is observed in the residuals.

4. **Residuals vs leverage**:

This plots finds the observations which have a high influence on the regression model according to the cooks distance. We can see that observation 42 is out of the region thus is an outlier which could have an impact on the model if we do remove or decide to keep it in the model.

#### 4.4.2 Model 2:Multiple Linear Regression

Create three new features for our multiple regression model: 

1. Total number of tokens involved per day:

First divide the total amount of each transaction with number of sub units of the storj token(10^8) to get the number of tokens, then we will group by and sum each amount according to the date  

First few records are as follows
```{r echo = FALSE}
newdataset$totalAmount <- (newdataset$totalAmount/(10^8))
tokentable <- aggregate(newdataset$totalAmount, by = list(Category = newdataset$Date), FUN = sum)
colnames(tokentable)<- c("Date","total_token")
newsimpletable <- merge(newsimpletable,tokentable,by = "Date")
head(newsimpletable$total_token)
```

2. Number of Unique buyers per day: 

Group the dataset according to the date and then count the number of unique buyers for each day.  

First few records are as follows:

```{r echo = FALSE}
buyerstable <- aggregate(data=newdataset, toNode ~ Date, function(x) length(unique(x)))
colnames(buyerstable) <- c("Date","unique_buyers")
newsimpletable  <- merge(newsimpletable,buyerstable,by = "Date")
head(newsimpletable$unique_buyers)
```

3. Percentage of investors who have bought more than 10 tokens:

Find the total number of investors per day and then find the total tokens bought by each investor per day. Then find the number of investors who bought more than 10 token that day.  

First few records are as follows:

```{r echo =FALSE ,message=FALSE ,error=FALSE}
library(dplyr)
investortable <- newdataset %>% group_by(Date)
totalinvestor_table<- summarize(investortable , total_investors = n_distinct(toNode))
newinvestortable <- newdataset %>% group_by(Date,toNode)
newinvestortable_totaltoken <- summarise(newinvestortable,total_token = sum(totalAmount))
subsetnewinvestortable_totaltoken <- newinvestortable_totaltoken[ which(newinvestortable_totaltoken$total_token>10), ]
finalinvestortable <- summarise(subsetnewinvestortable_totaltoken, final_investors = n_distinct(toNode))
newfinalinvestortable <- merge(totalinvestor_table,finalinvestortable,by = "Date")
newfinalinvestortable$percentage_investors <- (newfinalinvestortable$final_investors/newfinalinvestortable$total_investors)*100
newsimpletable <- merge(newsimpletable,newfinalinvestortable,by = "Date")
head(newsimpletable$percentage_investors)
```

Now that we have all the required features lets do the analysis as we did for the simple linear regression:

**Univariate analysis**:

Create a boxplot of unique buys regressor variable and then we check for the number of tokens(and capping at 94%) and percentage of investors variable:

```{r echo = FALSE,fig.cap = "Boxplot if variables for univariate analysis in multiple regression"}
par(mfrow=c(2,3))
bx = boxplot(newsimpletable$unique_buyers, main ="with outlier",ylab ="no of buyers")
#quantile(newsimpletable$unique_buyers, seq(0,1,0.02))
newsimpletable$unique_buyers<-ifelse(newsimpletable$unique_buyers>3087,3088,newsimpletable$unique_buyers)
boxplot(newsimpletable$unique_buyers,main ="without outliers",ylab = "no of buyers")
bx = boxplot(newsimpletable$total_token, main="with outlier", ylab ="no of tokens")
#quantile(newsimpletable$total_token, seq(0,1,0.02))
newsimpletable$total_token<-ifelse(newsimpletable$total_token>5791001,5791001,newsimpletable$total_token)
boxplot(newsimpletable$total_token, main="without outliers", ylab="no of tokens")
bx = boxplot(newsimpletable$percentage_investors, main="with outliers", ylab ="% of investors")
#quantile(newsimpletable$percentage_investors, seq(0,1,0.02))
newsimpletable$percentage_investors<-ifelse(newsimpletable$percentage_investors<60,61.2,newsimpletable$percentage_investors)
boxplot(newsimpletable$percentage_investors,main="without outliers", ylab="% of investors")
```


 After the univariate analysis lets do the **bivariate analysis**:
 
 plot number of transactions vs return:  
 plot total token vs return:  
 plot unique buyers vs return:  
 plot percentage investors vs return:  
 
```{r echo=FALSE, fig.cap = "Scatter plot of variables for Bivariate analysis in multiple regression"}
par(mfrow=c(2,2))
plot(newsimpletable$total_token,newsimpletable$return,xlab ="no of tokens",ylab ="simple return")
plot(newsimpletable$no_of_trans,newsimpletable$return,xlab ="no of trans",ylab ="simple return")
plot(newsimpletable$unique_buyers,newsimpletable$return,xlab ="no of buyers",ylab ="simple return")
plot(newsimpletable$percentage_investors,newsimpletable$return,xlab ="percentage of buyers",ylab ="simple return")
```
 
After analysing lets create an initial multiple regression linear model, Use the car library to calculate variable inflation factor and find the multicollinearity.
```{r error=FALSE ,message=FALSE}
final_data3 <- lm(return~no_of_trans+total_token+unique_buyers+percentage_investors,data = newsimpletable)
library(car)
vif(final_data3)
```

Since the VIF for all the regressor variables is not much higher than 5 we will keep all of them in the initial model

```{r}
summary(final_data3)
```

From the summary, p-value for the variables is very high suggesting that there is no linear relation between the regressor variable and the response variables.But before giving up on this we haven't yet used any features from the price data lets use the inherent features such as open, low, high etc to create a new linear model and see how we fare:

First find the multicollinearity and then use the step function to find the right combination of features:  

```{r echo=FALSE ,error =FALSE,message=FALSE,eval=TRUE}
final_data6 <- lm(return~no_of_trans+total_token+unique_buyers+percentage_investors+Open+High+Low,data = newsimpletable)
library(car)
vif(final_data6)
```

Finally, Check the summary and plot of our final data model:
```{r echo=FALSE,fig.cap = "Residual plot of the linear model for multiple regression"}
#anothernewsimpletable <- merge(priceData,freqtrans,by = "Date")
#summary(anothernewsimpletable)
#anothernewsimpletable<- anothernewsimpletable[,c(-6,-7)]
#shift <- function(x, n){
 # c(x[-(seq(n))], rep(NA, n))
#}
#anothernewsimpletable$return <- shift(anothernewsimpletable$return, 1)
#anothernewsimpletable <- anothernewsimpletable[1:(nrow(anothernewsimpletable)-1),]
#plot(anothernewsimpletable$no_of_trans,anothernewsimpletable$return)
#anothernewsimpletable <- merge(anothernewsimpletable,tokentable,by = "Date")
#anothernewsimpletable  <- merge(anothernewsimpletable,buyerstable,by = "Date")
#anothernewsimpletable <- merge(anothernewsimpletable,newfinalinvestortable,by = "Date")
#summary(anothernewsimpletable)
#cor(anothernewsimpletable[,c(2:12)])
#summary(final_data6)
final_data10 <- lm(return~total_token+Open+High+Low,data = newsimpletable)
#summary(final_data10)
#final_data8 <- lm(return~no_of_trans+total_token+unique_buyers+percentage_investors+High+Low,data = newsimpletable)
#summary(final_data8)
#vif(final_data8)
#final_data9 <- lm(return~no_of_trans+total_token+unique_buyers+percentage_investors+High,data = newsimpletable)
#vif(final_data9)
#summary(final_data9)
#final_data10 <- lm(return~total_token+Open+High+Low,data = newsimpletable)
summary(final_data10)
par(mfrow=c(2,2))
plot(final_data10)
```



## 5.Conclusion

### 5.1 Q1
We totally tried 5 distributions while fiting distributions with our data: Poisson Distribution, Weibull Distribution, Exponential Distribution, Geometric Distribution and Normal Distribution. After compare with parameters and graphs of different distributions, we can find:
The distribution of buyers and sellers are similiar.
The estimate parameters of Normal Distribution are very close to the parameters of our token data, but the graphs don't fit well. 
The graphs of Poisson Distribution and Weibull Distribution both fit our token data well, but for the parameters, the mean(buys:1331.66, sells:499.26) is not equal to variance(buys:133341824.3, sells:17815729.69), so it should not be Poisson Distribution.

Hence, we finally conclude that **Weibull Distribution** fit our distribution best.

### 5.2 Q2
The number of layers is 15. We tried several features of token data : (1) number of transactions(best correlation is 0.193) (2)Number of Unique buyers per day(best correlation is 0.14943) (3)Percentage of investors who have bought more than 10 tokens(best correlation is 0.1347)

Hence, the best correlation is 0.4943, when feature of token data is number of unique buyers per day.

### 5.3 Q3
For most active buyers, we tried 5 distributions: Poisson Distribution, Weibull Distribution, Exponential Distribution, Geometric Distribution, Normal Distribution.We also use ks-test to check which distribution fit our data best. After compare with parameters, graphs and p-value of different distributions, we can find: we get a bigest p-value = 0.3659 while fiting normal distribution, **normal distribution** fit our data best.
From above results and plot, we can find that the most active seller in our token (seller 6253227) only bought our token, cannot find purchase record in other tokens, so the distribution is just a line between Jul 2017 and Jan 2018, unable to fit any distribution.

### 5.4 Q4
When we use the price data columns and remove the features which have high p-value, we get $R^2$ value of **71%**. 
We have analysed how we can use simple and multiple linear regression to create linear models by doing feature extraction on the storj token and price data.
The model can be redeveloped and tested with different features to improve the R-squared value however the method of creation and analysis is mostly similar.
