~~~{r}
#https://github.com/sassoftware/R-swat/releases swat 패키지 다운로드(linux, osx, windows)
#아래와 같이 직접 인스톨 가능(osx 예)
install.packages('https://github.com/sassoftware/R-swat/releases/download/v1.4.0/R-swat-1.4.0-osx-REST-only.tar.gz',
                 repos=NULL,
                 type='file')

library(dplyr)
library(tidyverse)
library(swat)

# CSV 파일 읽기
dat_2017 <- read.csv('2017-nego.csv',header = TRUE, fileEncoding = "cp949")
dat_2018 <- read.csv('2018-nego.csv',header = TRUE, fileEncoding = "cp949")
dat_2019 <- read.csv('2019-nego.csv',header = TRUE, fileEncoding = "cp949")


# YEAR 컬럼 추가
dat_2017 = mutate(dat_2017,YEAR='2017')
dat_2018 = mutate(dat_2018,YEAR='2018')
dat_2019 = mutate(dat_2019,YEAR='2019')

# 데이터 합치기
data_all <- rbind(dat_2017,dat_2018,dat_2019)

# 컬럼명 변경
col_name_list <-  c("사업부코드","사업부명","호선번호","엔진타입","계약번호","공사번호","SER","SEQ","자재번호","품명","도면번호","계약일자","수량단위","계약수량","중량단위",
                 "계약중량","품종코드","계약업체코드","계약업체명","계약화폐단위","계약단가","계약단가(USD)","계약단가(KRW)","계약금액","계약금액(USD)","계약금액(KRW)",
                 "구매팀","구매IM","구매담당","견적번호","견적분할번호","PI화폐단위","PI금액","PI금액(USD)","PI금액(KRW)","견적업체코드","견적업체명","견적화폐단위",
                 "최초견적금액","최초견적금액(USD)","최초견적금액(KRW)","최종견적금액","최종견적금액(USD)","최종견적금액(KRW)","견적계약업체코드","견적계약화폐단위","견적계약금액",
                 "견적계약금액(USD)","견적계약금액(KRW)",'YEAR')

for (i in 1:length(col_name_list)){
  names(data_all)[i] <- col_name_list[i]
}
colnames(data_all)

??drop_na

# nego_ratio 컬럼생성 하고 NEGO_RATIO NaN 제거
# data_all2 <- data_all %>% 
#   mutate(네고율 = round((최초견적금액 - 최종견적금액) / 최초견적금액 * 100,2)) %>%
#   drop_na(네고율)
  
#
data_all_add_nego <- data_all %>% 
  drop_na(최초견적금액) %>%
  drop_na(최종견적금액) %>%
  subset(최초견적금액 > 0) %>%
  subset(최종견적금액 > 0) %>%
  mutate(네고율 = round((최초견적금액 - 최종견적금액) / 최초견적금액 * 100,2)) %>%
  drop_na(네고율) %>%
  subset(네고율 > 0 & 네고율 < 100)


view(data_all_add_nego)


# 특정 조건의 Row 제거 하기(일단 네고율이 100 인것만 제거함)
#subset(data_all2, 네고율 >= 0 & 최종견적금액 > 0 & 최초견적금액 > 0)
#(data_detail_filter <- filter(data_all2, 네고율 >= 0, 최종견적금액 > 0 ,최초견적금액 > 0))

#data_detail <- data_all2[(data_all2$네고율 != 100),]

# 집계 데이타 생성 (max,mean,min)
data_agg <- summarise(group_by(data_all_add_nego, YEAR, 사업부코드, 계약업체코드), max=round(max(네고율),2), mean=round(mean(네고율),2), min=round(min(네고율),2))


# write to csv file in local system
Sys.setlocale(locale="cp949")
encodeConnection <- file('nego_detail_r.csv', encoding = "UTF-8")
write.csv(data_all_add_nego, file = encodeConnection)

encodeConnection <- file('nego_agg_r.csv', encoding = "UTF-8")
write.csv(data_agg, file = encodeConnection)


# SAS Viya 접속  
# window, linux 의 경우 5570(Binary) 8777(text) 2가지 모드 접속 가능
# osx 의 경우 8777(text) 모드로만 접속 가능
# Viya 서버의 '/opt/sas/viya/config/etc/SASSecurityCertificateFramework/cacerts/trustedcerts.pem' 파일 다운로드
Sys.setenv(CAS_CLIENT_SSL_CA_LIST = "/Users/dangtong/Dropbox/05.Dev/11.R/hyundai/trustedcerts.pem")
Sys.getenv("CAS_CLIENT_SSL_CA_LIST")
conn <- CAS('127.0.0.1', 8777,protocol='https', username='sasdemo09',password='demopw')
res <- cas.builtins.serverStatus(conn) # 서버 상태 확인
res 

# data_detail, data_agg 를 Viya CAS 테이블에 업로드
cas.upload.frame(conn, data_agg,  casOut = "NEGO_AGGS")

####최초 업로드시 
agg_ct <- as.casTable(conn, data_agg, casOut=list(name="data_agg2", caslib='PUBLIC', promote=TRUE))
detail_ct <- as.casTable(conn, data_all_add_nego, casOut=list(name="data_detail2", caslib='PUBLIC', promote=TRUE))

#테이블 갱신시
agg_ct <- as.casTable(conn, data_agg, casOut=list(name="data_agg2", caslib='PUBLIC',replace=TRUE))
agg_ct <- as.casTable(conn, data_all_add_nego, casOut=list(name="data_detail2", caslib='PUBLIC',replace=TRUE))



~~~

