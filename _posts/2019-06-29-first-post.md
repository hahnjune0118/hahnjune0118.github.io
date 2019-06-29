---
title: "Welcome to Jekyll!"
date: 2019-06-29 12:25:28 -0400
categories: jekyll update
---
---
title: "A Linear Regression Analysis on Apartment Prices in Seoul"
subtitle: "서울시 아파트 가격에 대한 선형회귀 분석"
author: "HahnJune Yeon"
date: "2019.06.29"
output: html_document
---
version: 5.0

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## About this Project

이 문서는 RMarkdown에서 작성했습니다. 해당 프로젝트는 한양대학교 <a href = "http://ctl.hanyang.ac.kr/home">R.PBL교수학습센터</a>에서 2019학년도 1학기 동안 진행한 **글로벌학습공동체:공강** 활동의 결과물입니다.

한양대학교 경제금융학과 2학년에 재학중인 4명의 학생들로 구성된 `또래오래` 팀은 계량경제(ECO3007) 과목에서 배운 내용을 실제 데이터에 적용해보기 위해서 이 프로젝트를 진행했습니다. 해당 문서는 팀장 연한준(경금,18)이 작성했습니다.

## Tools and  Data

데이터 분석은 오픈소스 통계 프로그램인 R <https://www.r-project.org/> 과 개발환경(IDE)으로는 R-Studio <https://www.rstudio.com/> 를 이용했습니다. 

부동산 데이터는 문재인 정부가 출범한 2017년 5월 이후 **2017년 6월 16일부터 2019년 6월 14일까지** 2년간의 서울시 아파트 매매 가격을 사용했습니다. 부동산 데이터는 '국토교통부 실거래가 공개시스템' <http://rtdown.molit.go.kr/>을 활용했습니다. 


## Initial Package Settings

```{r package, warning=FALSE}
install.packages('readxl', repos = "https://cran.r-project.org/")
library(readxl)
install.packages('ggplot2', repos = "https://cran.r-project.org/")
library(ggplot2)
install.packages('tidyverse', repos = "https://cran.r-project.org/")
library(tidyverse)
install.packages('dplyr', repos = "https://cran.r-project.org/")
library(dplyr)
install.packages('reshape2', repos = "https://cran.r-project.org/")
library(reshape2)
```


## Data Combining

##### 한번에 최대 1년치의 데이터만을 다운 받을 수 있어 2개의 파일을 병합했습니다.
##### .xlsx 파일을 불러오기 전, 데이터에 관한 설명 부분을 Excel을 이용해 삭제해 전처리 과정을 단순화했습니다. 

```{r comb, warning=FALSE}
library(readxl)
data18 <- read_excel('아파트(매매)_실거래가_201706-201806.xlsx')
data19 <- read_excel('아파트(매매)_실거래가_201806-201906.xlsx')
datac<-rbind(data18,data19)
```


## Data Labelling

```{r label, warning=FALSE, echo=TRUE, results = 'hide'}
library(dplyr)
## '시군구'라는 한개의 변수로 저장된 데이터를 서울시에 맞춰 각각 '시' '구' '동'의 변수로 분할
datas<-separate(datac, col=시군구, sep=' ', into=c('시','구','동'))

## 한글로 저장된 변수를 간단한 영어로 변환
ym<-datas$계약년월
si<-datas$시
gu<-datas$구
dong <- datas$동
bunji <- datas$번지
danji <- datas$단지명

## string(문자형)으로 저장된 변수를 numeric(숫자형)으로 변환
floor <- datas$층
floor <- as.numeric(floor)
const_year <- datas$건축년도
const_year <- as.numeric(const_year)
livingArea <- datas$`전용면적(㎡)`
livingArea <- as.numeric(livingArea)

## 제곱미터로 표현된 전용면적을 평 단위로 변환
livingArea_KR <- livingArea/3.30579

## 1) 변수 이름 변경
price<-datas$`거래금액(만원)`
## 2) 천단위 마다 표시된 콤마 삭제
pricey<-gsub(",","", price)
## 3) 문자형 변수로 변환
price<- as.numeric(pricey)
## 4) 단위를 만원에서 백만원 단위로 변환
price100<-price/100
price100
```


## Data Fromatting

```{r format, warning=FALSE}
## 문자형으로 저장된 거래년월 데이터를 숫자형으로 변환
ym<-as.numeric(ym)
## 년과 월 사이에 '-' 삽입
ymDate<-gsub('^(.{4})(.*)$', '\\1-\\2', ym)
## 계약일 변수를 숫자형으로 변환
day<-as.numeric(datas$계약일)
## 계약년월과 일 변수를 '-'로 이어 새로운 변수 생성
ymd = paste(ymDate,day,sep="-")
## 숫자형 변수를 날짜 형태로 변환
ymd_date<- as.Date(ymd)

## 계약 날짜, 시, 구, 동, 아파트 번지, 단지, 층수, 건축년도, 전용면적, 가격으로 구성된 새로운 2차원 구조의 data.frame 생성
houseData = data.frame(ymd_date, si, gu, dong, bunji, danji, 
                       floor, const_year, livingArea_KR, price100)
```


## Speculation Labelling

#### 문재인 정부 출범(2017년 5월 이후) 첫 대규모 부동산 대책인 **8.2 부동산 대책** <a href = "http://www.molit.go.kr/USR/NEWS/m_71/dtl.jsp?id=95079498">국토교통부 보도자료</a> 의 핵심인 **투기지역, 투기과열지구** 지정에 따른 서울시 아파트 매매가격의 변화 및 차이를 분석했습니다. 
##### 투기지역과 투기과열지구 선정에 따른 각각의 정책은 보도자료에서 확인할 수 있습니다.
```{r spec, warning=FALSE}
## 서울시 전역이 투기과열지구로 지정
## 투기지역은 전체 25개 구 중 해당 11개에 해당
spec<-ifelse(houseData$gu=='강남구'|houseData$gu=='서초구'|houseData$gu=='송파구'
                       |houseData$gu=='강동구'|houseData$gu=='용산구'|houseData$gu=='성동구'
                       |houseData$gu=='노원구'|houseData$gu=='마포구'|houseData$gu=='양천구'
                       |houseData$gu=='영등포구'|houseData$gu=='강서구',1,0)
## 기존의 data.frame에 투기지역 선정 여부 변수(더미변수) 추가
houseData = data.frame(ymd_date, si, gu, dong, bunji, danji, 
                       floor, const_year, livingArea_KR, price100, spec)
## data.frame의 구조 열람
str(houseData)
```


#### 투기지역(=1)과 투기조정지역(=0)의 아파트 매매가격 차이 
```{r specgraph, warning=FALSE}
install.packages('ggpubr', repos = "https://cran.r-project.org/")
library(ggpubr)
ggboxplot(houseData, x='spec', y='price100',
          color='spec', palette = c("#00AFBB", "#FC4E07"),
          xlab='Speculation', ylab='Price')
```


## Apartment Branding

### 아파트 브랜드가 가격에 미치는 영향 분석
#### 아파트 선호 브랜드 상위 10개는 부동산114와 한국리서치가 2018년 11월 전국 성인 남녀 5049명을 대상으로 조사한 '2018년 베스트 아파트 브랜드' 설문조사를 참고했습니다.
```{r brand, warning=FALSE}
houseData <- mutate(houseData, premium = ifelse(grepl('래미안|편한세상|푸르지오|자이|더샵|
                     힐스테이트|롯데캐슬|아이파크|위브|VIEW', houseData$danji),1,0))
## 전체 아파트 매매 데이터 중 고급 브랜드에 해당하는 건수: 23,545
## 전체 아파트 매매 데이터 건수: 146,566

#### 15만개에 달하는 매매가격 데이터를 처리할 컴퓨팅 능력이 부족해 단순 무작위 비복원추출을 사용해 데이터를 경량화했습니다.
houseDataSample <- sample_n(houseData, 1000)
```

## Interactive Bubble Chart
#### 지역구에 따른 아파트 건설년도를 시각화해보았습니다. 2017년부터 2019년에 거래된 아파트는 대체적으로 1990년에서 2010년 사이 지어진 아파트가 많다는 것을 볼 수 있습니다.
```{r bubble, warning=FALSE}
install.packages('plotly', repos = "https://cran.r-project.org/")
library(plotly)
bub <- plot_ly(houseDataSample, x = ~const_year, y = ~gu, text = ~price100, 
               type = 'scatter', mode = 'markers')
bub
```

## Linear Regression

#### 1.대지면적(평수) 2.투기지역 선정 여부 3.건설시기 4.층수 5.선호되는 아파트인지의 여부 의 총 5개의 변수가 아파트 가격에 미치는 영향을 선형회귀모형으로 분석했습니다.

#### 선형회귀분석은 종속변와 독립변수들 간의 선형관계를 나타내는 식으로, $$y_i = \alpha + \beta_1 x_i1 + ... + \beta_p x_ip + \epsilon_i,   i = 1,...,n$$의 형태로 표현됩니다. 
##### 선형회귀분석에 대한 간략한 설명은 <a href = "https://blog.naver.com/istech7/50152984368">여기</a>에서 보실 수 있습니다.
```{r reg, warning=FALSE}
model1 <- lm(price100 ~ livingArea_KR + spec + const_year + floor + premium
             ,data=houseDataSample)

## 각 변수의 단위가 1만큼 증가할 때 평균적으로 변하는 아파트 가격
## ex) floor: 아파트 층수가 한단위 높아지면 평균적으로 매매가는 4.891백만원 증가한다.
model1$coef[1]

## 회귀분석으로 출력된 각 변수들의 영향력(coefficient)의 신뢰치
## 건설년도(const_year)를 제외한 모든 변수의 p-value (Pr(>|t|))가 0에 가깝습니다. 즉 99% 이상의 신뢰수준을 가집니다. 
## 신뢰수준 99%란, 신뢰구간을 구하는 일을 무한히 반복할때 99%의 경우엔 신뢰구간안에 모집단(조사대상)의 평균이 있다는 뜻입니다.

## 해당 선형모형의 결정계수(R-sqaured)는 0.5046로, 5개의 변수가 아파트 가격 변화의 50.46%를 설명한다는 의미를 가집니다.
summary(model1)
```

## Linear Regression Model Diagnostics

#### 1.Residual vs Fitted: X 축에 선형 회귀로 예측된 Y 값, Y 축에는 잔차를 보여준다. 선형 회귀에서 오차는 평균이 0이고 분산이 일정한 정규 분포를 가정하였으므로, 예측된 Y 값과 무관하게 잔차의 평균은 0이고 분산은 일정해야 한다. 따라서 이 그래프에서는 기울기 0인 직선이 관측되는 것이 이상적이다.
#### 2. Normal Q-Q: 잔차가 정규 분포를 따르는지 확인하기 위한 그래프이다.
#### 3. Scale-Location: X 축에 선형 회귀로 예측된 Y 값, Y 축에 표준화 잔차(Standardized Residual)를 보여준다. 이 경우도 기울기가 0인 직선이 이상적이다. 만약 특정 위치에서 0에서 멀리 떨어진 값이 관찰된다면 해당 점에 대해서 표준화 잔차가 크다, 즉, 회귀 직선이 해당 Y를 잘 적합하지 못한다는 의미다. 이런 점들은 이상치(outlier)일 가능성이 있다.
#### 4. Residuals vs Leverage: X 축에 레버리지(Leverage), Y 축에 표준화 잔차를 보여준다. 레버리지는 설명 변수가 얼마나 극단에 치우쳐 있는지를 뜻한다. 예를 들어, 다른 데이터의 X 값은 모두 1 ~ 10 사이의 값인데 특정 데이터만 99999 값이라면 해당 데이터의 레버리지는 큰 값이 된다. 이런 데이터는 입력이 잘못되었거나, 해당 범위의 설명 변숫값을 가지는 데이터를 보충해야 하는 작업 등이 필요하므로 유심히 살펴봐야 한다.
##### 네 번째 차트의 우측 상단과 우측 하단에는 선으로 쿡의 거리Cook’s Distance가 표시되어 있다. 쿡의 거리는 회귀 직선의 모양(기울기나 절편 등)에 크게 영향을 끼치는 점들을 찾는 방법이다. 쿡의 거리는 레버리지와 잔차에 비례하므로 두 값이 큰 우측 상단과 우측 하단에 쿡의 거리가 큰 값들이 위치하게 된다.
###### 출처: 
<https://thebook.io/006723/ch08/02/06/>

<http://www.sthda.com/english/articles/39-regression-model-diagnostics/161-linear-regression-assumptions-and-diagnostics-in-r-essentials/>

```{r diag, warning=FALSE}
par(mfrow=c(2,2))
plot(model1)
```

## Single Varibale Linear Model

### 대지면적과 아파트 가격의 선형관계
```{r linear, warning=FALSE}
par(mfrow=c(1,1))
plot(price100 ~ livingArea_KR, houseDataSample)
model2 <- lm(price100 ~ livingArea_KR, houseDataSample)
model2
abline(model2, col='red')
```

## Linear Model with ggplot2 package
```{r gg, warning=FALSE}
housegg <- p<-ggplot(data=houseDataSample,aes(x=livingArea_KR,y=price100))+geom_point()
p+geom_abline(intercept = coef(model2)[1], slope = coef(model2)[2],col="red")

##  신뢰구간을 포함한 선형회귀 모델링
p+stat_smooth(method=lm)

## 통계량
model3 <- lm(price100 ~ livingArea_KR * premium, data = houseData)
summary(model3)
```

## Linear Modelling by Apartment Brand
### 프리미엄 아파트 프랜드에 따른 각각의 선형회귀 모델
```{r ggg, warning=FALSE}
p<-ggplot(data=houseDataSample,aes(x=livingArea_KR,y=price100,colour=(as.logical(premium)))) + 
  geom_point()+geom_smooth(method="lm")
p
```

## Alternative Method
```{r alt, warning=FALSE}
model4 <- lm(price100 ~ livingArea_KR + premium, data = houseData)

equation1=function(x){coef(model4)[2]*x+coef(model4)[1]}
equation2=function(x){coef(model4)[2]*x+coef(model4)[1]+coef(model4)[3]}

library(ggplot2)
g <- ggplot(houseDataSample,aes(y=price100,x=livingArea_KR,colour=(as.logical(premium))))+
  geom_point()+
  stat_function(fun=equation1,geom="line",color=scales::hue_pal()(2)[1])+
  stat_function(fun=equation2,geom="line",color=scales::hue_pal()(2)[2])
g

```


#### Thank You!
