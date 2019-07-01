## A Linear Regression Analysis on Apartment Prices in Seoul
### 서울시 아파트 가격에 대한 선형회귀 분석
### HahnJune Yeon
#### last updated: 2019.06.29 22:00

### About this Project
이 문서는 RMarkdown에서 작성했습니다. 
해당 프로젝트는 한양대학교 [R.PBL교수학습센터](http://ctl.hanyang.ac.kr/home)에서 
2019학년도 1학기 동안 진행한 **글로벌학습공동체:공강** 활동의 결과물입니다.

한양대학교 경제금융학과 2학년에 재학중인 4명의 학생들로 구성된 **또래오래** 팀은 
`계량경제(ECO3007)`[(커리큘럼)](https://portal.hanyang.ac.kr/openPop.do?header=hidden&url=/haksa/SughAct/findSuupPlanDocHyIn.do&flag=DN&year=2019&term=10&suup=11576&language=ko#!Q09OVFJPTExFUiRAXiNoeWluQ29udGVudHMkQF4kQF5oYWtzYS9vcGVuUGFnZS5kbz9wZ21JZD1QMzIwNjY0JnRrPTgzZTgxYzY2MTdiMjBhYzAzNDQ5NDQwNDAxN2YyM2Q3Mzk3YTYzZTE3YmNjZTE4ODNlZGU0NmZhMTE5NTEwMzEmcEZsYWc9RE4mcFN1dXBZZWFyPTIwMTkmcFN1dXBUZXJtPTEwJnBTdXVwTm89MTE1NzYmcENvdXJzZUlkPSZwR2FlaW5Obz0mcExvY2FsZT1rbw==) 
과목에서 배운 내용을 실제 데이터에 적용, 응용해보기 위해 이 프로젝트를 진행했습니다. 
해당 문서는 팀장 `연한준(경금,18)`이 작성했습니다.

### Tools and Data
데이터 분석은 오픈소스 통계 프로그램인 R(https://www.r-project.org/)과 
개발환경(IDE)으로는 R-Studio(https://www.rstudio.com/)를 이용했습니다.

부동산 데이터는 문재인 정부가 출범한 2017년 5월 이후 **2017년 6월 16일**부터 **2019년 6월 14일**까지 2년간의 서울시 아파트 매매 가격을 사용했습니다. 
부동산 데이터는 `국토교통부 실거래가 공개시스템`(http://rtdown.molit.go.kr/)을 활용했습니다.

### Initial Package Settings
```markdown
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

### Data Combining
한번에 최대 1년치의 데이터만을 다운 받을 수 있어 2개의 파일을 병합했습니다.

.xlsx 파일을 불러오기 전, 데이터에 관한 설명 부분을 Excel을 이용해 삭제해 전처리 과정을 단순화했습니다.

```markdown
library(readxl)
data18 <- read_excel('아파트(매매)_실거래가_201706-201806.xlsx')
data19 <- read_excel('아파트(매매)_실거래가_201806-201906.xlsx')
datac<-rbind(data18,data19)
```

### Data Labelling
```markdown
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

### Data Fromatting
```markdown
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







```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```
