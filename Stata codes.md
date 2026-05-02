자료를 다운로드 받지 않고 사용하려면 Stata에서 예를 들어 다음과 같이
명령함(마지막 `testfe`를 적절한 파일로 바꿈)

```stata
global datadir "https://github.com/chan079/panelbook/raw/main/data"
use $datadir/testfe, clear
```

# 1. 도입

## 1.1 Stata 연습

### 연습 1.4

```stata
set more off
cd "c:/Documents/Data Folder"
log close _all
log using ex.smcl, replace
*** Work here ***
log close
set more on
```

## 1.2 계량경제학의 기초

### 연습 1.8

```stata
use death1, clear
reg deathrate smoke if year==2010
```

### 연습 1.9

아래에서 `*` 다음은 코멘트. `*` 대신에 `//`를 사용하면 `do` 파일에서는
괜찮지만 직접 명령창에 입력하면 오류 발생.

```stata
* continued
reg deathrate smoke aged if year==2010
```

### 연습 1.10

```stata
* continued
reg deathrate smoke aged i.year, vce(cl region)
```

### 연습 1.12

```stata
use hprice1, clear
reg lprice bdrms colonial
reg lprice bdrms colonial lsqrft
```

### 연습 1.13

```stata
gen lbdrmsize = ln(sqrft/bdrms)
reg lprice bdrms colonial lbdrmsize
```

### 연습 1.14

```stata
use mlb1, clear
reg lsalary years gamesyr bavg hrunsyr rbisyr
su years gamesyr bavg hrunsyr rbisyr
```

### 연습 1.15

```stata
use wage2, clear
reg lwage educ exper tenure married black south urban
```

### 연습 1.16

```stata
* continued
reg lwage educ exper tenure married black south urban IQ
```

### 연습 1.17

```stata
* continued
reg lwage educ exper tenure married black south urban KWW
```

### 연습 1.18

```stata
* continued
reg lwage educ exper tenure married black south urban IQ KWW
```

### 연습 1.19 앞

```stata
use wage2, clear
reg hours lwage age married black
```

### 연습 1.21 앞

```stata
use death1, clear
reg deathrate drink smoke aged
reg deathrate drink smoke aged, vce(r)
reg deathrate drink smoke aged, vce(cl region)
```

### 연습 1.23

```stata
use death1, clear
reg deathrate drink smoke aged i.year
reg deathrate drink smoke aged i.year, vce(r)
reg deathrate drink smoke aged i.year, vce(cl region)
```

### 연습 1.25

아래 `do` 파일 내용에서 `//` 다음은 코멘트 문이다. `//`를 명령창에
직접 입력하면 오류가 발생하므로 이를 제외하고 복사.

```stata
import excel using munnell90.xls, firstrow clear
xtset ... // HERE

label var st_abb "state abbreviation"
label var state "state (numeric)"
label var yr "year from 1970 to 1986"
label var p_cap "public capital"
label var hwy "highway capital"
label var water "water utility capital"
label var util "utility capital"
label var pc "private capital stock"
label var gsp "gross state product"
label var emp "employment in non-agricultural payrolls"
label var unemp "state unemployment rate (%)"

foreach v of varlist gsp hwy water util pc emp {
  gen ln`v' = ln(`v')
}

save munnell90, replace
```

# Part I 선형패널모형

# 2. 선형패널모형과 추정

## 2.1 선형패널모형

## 2.2 집단 간 회귀, 통합 회귀, 집단 내 회귀

### 연습 2.2 앞

```stata
use gasoline, clear
xtset country year
d
xtreg lgaspcar lincomep lrpmg lcarpcap, be
```

### 연습 2.4

```stata
use wdi5bal, clear
xtreg sav age0_19 age20_29 age65over lifeexp i.year, be
xtdescribe
use wdi5data, clear
xtreg sav age0_19 age20_29 age65over lifeexp i.year, be
xtdescribe
```

### 연습 2.6 앞

```stata
use gasoline, clear
reg lgaspcar lincomep lrpmg lcarpcap i.year
```

### 연습 2.7

```stata
reg lgaspcar lincomep lrpmg lcarpcap, vce(cl country)
```

### 연습 2.8 앞

```stata
use gasoline, clear
xtreg lgaspcar lincomep lrpmg lcarpcap i.year, fe
```

### 연습 2.13 앞

```stata
use gasoline, clear
foreach v of varlist lincomep lrpmg lcarpcap {
  by country: egen bar_`v' = mean(`v')
}
qui reg lgaspcar lincomep lrpmg lcarpcap bar_*
est store ols
qui xtreg lgaspcar lincomep lrpmg lcarpcap bar_*, re
est store re
qui xtreg lgaspcar lincomep lrpmg lcarpcap, fe
est store fe
qui xtreg lgaspcar lincomep lrpmg lcarpcap, be
est store be
est tab ols re fe be, b se
```

### 연습 2.16

```stata
clear all
use small-dataset, clear
xtset id year
list, sep(4)
xtreg y x z, fe
foreach v of varlist x z {
  by id: egen bar_`v' = mean(`v')
}
reg y x z bar_*
by id: egen y0 = total(y / (year==0)), missing
l, sep(4)

levelsof year, local(yearlevels)
foreach v of varlist x z {
  foreach year of local yearlevels {
    by id: egen `v'_`year' = total(`v' / (year == `year')), missing
  }
}
drop *_0
reg y x z x_* z_*
```

## 2.3 임의효과 모형과 고정효과 모형

### 연습 2.17 앞

```stata
use gasoline, clear
xtreg lgaspcar lincomep lrpmg lcarpcap i.year, re
```

### 연습 2.23

```stata
use death1, clear
foreach v of varlist deathrate smoke aged {
  gen ln`v' = ln(`v')
}
reg lndeathrate lnsmoke lnaged i.year, vce(cl region)
xtreg lndeathrate lnsmoke lnaged i.year, re vce(r)
```

### 연습 2.24

```stata
* continued
reg d.(lndeathrate lnsmoke lnaged) i.year, vce(cl region)
xtreg lndeathrate lnsmoke lnaged i.year, fe vce(r)
```

### 연습 2.25

```stata
use death1, clear
drop if year==2008
reg d.(deathrate smoke aged), nocons
reg d.(deathrate smoke aged)
xtreg deathrate smoke aged, fe
xtreg deathrate smoke aged i.year, fe
```

### 연습 2.26

```stata
* continued
reg d.(deathrate smoke aged) i.year
```

### 연습 2.27 앞

```stata
use gasoline, clear
xtreg lgaspcar lincomep lrpmg lcarpcap i.year, fe
areg lgaspcar lincomep lrpmg lcarpcap i.year, a(country)
```

### 연습 2.29 앞

```stata
use testfe, clear
xtreg y x1 x2, fe
est store fe
xtreg y x1 x2 z1, re
hausman fe .
```

### 연습 2.30

```stata
use hausman-odd, clear
xtreg y x1 x2, fe
est store fe
xtreg y x1 x2 z1, re
hausman fe .
```

### 연습 2.31

```stata
use testfe, clear
xtreg y x1 x2, fe vce(r)
est store fe
xtreg y x1 x2, re vce(r)
est store re
hausman fe re
```

### 연습 2.32

```stata
use testfe, clear
foreach v of varlist x1 x2 {
  by id: egen bar_`v' = mean(`v')
}
// xtreg y x1 x2 bar_*, re
reg y x1 x2 bar_*, vce(cl id)
testparm bar_*
// xtreg y x1 x2 z1 bar_*, re
reg y x1 x2 z1 bar_*, vce(cl id)
testparm bar_*
```

### 연습 2.33

```stata
xtreg y x1 x2 z1, be
```

### 연습 2.37

```stata
use munnell90, clear
global model lngsp lnhwy lnwater lnutil lnpc lnemp unemp
reg $model i.yr
reg $model i.yr, vce(r)
reg $model i.yr, vce(cl state)
xtreg $model i.yr, re
xtreg $model i.yr, re theta
xtreg $model i.yr, re vce(r)
reg d.($model) i.yr, vce(cl state)
xtreg $model i.yr, fe
xtreg $model i.yr, fe vce(r)
predict xb, xb
predict xbu, xbu
twoway line lngsp xb xbu yr if state==1
xtreg $model i.yr, be

qui xtreg $model i.yr, re
est store re
qui xtreg $model i.yr, fe
hausman . re, sigma

foreach v of varlist lnhwy lnwater lnutil lnpc lnemp unemp {
  by state: egen `v'_bar = mean(`v')
}
reg $model *_bar i.yr, vce(cl state)
testparm *_bar

xtreg $model *_bar i.yr, re vce(r)
testparm *_bar
```

### 연습 2.38

```stata
use ict, clear
gen lsales = ln(sales)
gen lemp = ln(emp)
gen lcap = ln(cap)
gen kospi = market==1
qui reg lsales lcap lemp foreign kospi i.sector
bysort id: egen nobs = sum(e(sample))
keep if nobs == 12
drop nobs
xtset id year
xtsum
xtsum sector
tab sector if year==2005
xtsum kospi
tab kospi if year==2005

xtreg lsales lcap lemp foreign i.year, fe
est store fe
xtreg lsales lcap lemp foreign kospi i.sector i.year, re
hausman fe ., sigma
xtreg lsales lcap lemp foreign kospi i.sector, be

foreach v of varlist lemp lcap foreign {
  by id: egen `v'_bar = mean(`v')
}
reg lsales lcap lemp foreign kospi i.sector i.year *_bar, vce(cl id)
testparm *_bar
```

## 2.4 Population-Averaged 모형

### 연습 2.39 앞

다음에서는 `local` 매크로를 사용하므로 `do` 파일에서만 작동. 명령창에서 복사/붙여넣기로 사용하려면 매크로를 `local` 대신에 `global`로 정의하고 `$model`과 같이 사용.

```stata
use gasoline, clear
local model "lgaspcar lincomep lrpmg lcarpcap"
reg `model'
xtreg `model', pa c(ind)
xtreg `model', re
xtreg `model', pa c(exc)
```

## 2.5 강외생성하 추정과 관련된 심화주제들

### 연습 2.44 앞

```stata
use gasoline, clear
local model "lgaspcar lincomep lrpmg lcarpcap"
qui xtreg `model', be
est store be
qui xtreg `model', fe
est store fe
qui xtreg `model', re
est store re
qui reg `model'
est store pols
est tab fe pols re be, b se stats(r2 r2_w r2_o r2_b)
```

### 2.5.2절 시작 부분

```stata
use didex, clear
xtreg y d, fe vce(r)
xtreg y d i.year, fe vce(r)
```

### 연습 2.49 앞 (2.5.2절 두 번째 부분)

```stata
use didex, clear
reg y part##after, vce(cl id)
```

### 연습 2.51

```stata
use didexunb, clear
xtdes
xtreg y d i.year, fe
```

### 연습 2.52

앞에서 계속

```stata
reg y part##after, vce(cl id)
xtreg y d i.year, fe vce(r)
```

### 연습 2.57

```stata
webuse hospdd, clear
didregress (satis) (procedure), group(hospital) time(month)
estat trendplots, omeans
estat ptrends
estat grangerplot
gen trgrp = hospital <= 18
gen mon1 = month-0.06
gen mon2 = month+0.06
twoway (scatter satis mon1 if !trgrp) ///
  (scatter satis mon2 if trgrp & procedure==1) ///
  (scatter satis mon2 if trgrp & procedure==0)
```

위에서 `///`는 줄바꿈을 뜻함. `do` 파일에서는 문제가 없으나, 명령창에 직접 입력할 때에는 `///`와 줄바꿈을 없애고 한 줄로 할 것. `twoway (scatter satis mon1 if !trgrp) (scatter satis mon2 if trgrp & procedure==1) (scatter satis mon2 if trgrp & procedure==0)`

### 연습 2.58 앞

```stata
use https://friosavila.github.io/playingwithstata/drdid/mpdta.dta, clear
table year first
xtset county year
gen d = treat & year >= first_treat
xtdidregress (lemp) (d), group(countyreal) time(year)
estat bdecomp
xthdidregress aipw (lemp lpop) (d lpop), group(countyreal)
estat aggreg, dynamic
```

### 연습 2.66 앞

아래에서 줄 마지막의 `///`는 ‘다음 줄로 계속됨’을 뜻한다. `do`
파일 내에서는 작동하나 직접 명령창에 입력하면 오류가 발생할 것이다.
직접 명령창에 입력하려면 `///` 없이 한 줄에 입력해야 한다.

```stata
use psidextract, clear
xthtaylor lwage wks south smsa ms exp exp2 occ ind union fem blk ed, ///
   endog(exp exp2 occ ind union ed)
```

# 4. 선형 동적 패널 모형의 추정

## 4.1 선형 동적 패널 모형

## 4.2 고정효과 동태적 패널 모형의 GMM 추정

### 연습 4.4 다음, 연습 4.5 앞

```stata
use ajry08five, clear
xtabond dem yr3-yr11 if sample==1, pre(inc_1) vce(r) nocons
```

위 내용을 여기서 복사/붙여넣기 하지 않고 pdf 파일에서 복사/붙여넣기 하면 무슨 일이 발생하는가?

### 연습 4.7 앞

```stata
use ajry08five, clear
qui xtabond dem yr3-yr11 if sample==1, pre(inc_1) nocons two
estat sargan
```

### 연습 4.8 앞

```stata
use ajry08five, clear
qui xtabond dem yr3-yr11 if sample==1, pre(inc_1) nocons two
estat abond
```

### 연습 4.14 앞

```stata
set more off
clear all
local n 5000
local T 10
set obs `=`n'*`T''
gen id = ceil(_n/`T')
by id, sort: gen year = 1990 + _n
xtset id year
set seed 1
gen e = rnormal()
by id: gen y = sum(e)  // random walk for each i
save unitroot, replace
reg l(0/1).y
ivregress 2sls d.y (ld.y = l2.y), vce(cl id) first
xtabond y
set more on
```

## 4.3 System GMM

### 4.3.2절 (연습 4.16 앞)

```stata
use unitroot, clear
xtabond y
xtdpdsys y
```

### 연습 4.18 앞

```stata
use ajry08five, clear
xtdpdsys dem yr3-yr11 if sample==1, pre(inc_1) vce(r) nocons
```

### 연습 4.21

```stata
use growth, clear
gen y = ln(gdp)
gen s = ln(saving)
gen n = ln(pop)
qui tab year, gen(yr)
xtabond y n yr3-yr26, pre(s) vce(r)
estat abond
xtabond y n yr3-yr26, pre(s) two vce(r)
estat abond
ivregress 2sls d.(y n) yr4-yr26 (ld.y d.s = l2.y l.s), vce(cl id)
xtdpdsys y n yr3-yr26, pre(s) vce(r)
xtdpdsys y n yr3-yr26, pre(s)
estat sargan
xtdpdsys y n yr3-yr26, pre(s) two
estat sargan
```

### 연습 4.22

```stata
use growth, clear
gen y = ln(gdp)
gen s = ln(saving)
gen n = ln(pop)
qui tab year, gen(yr)
xtabond y n yr3-yr26, pre(s) vce(r)
estat sargan
estat abond
xtabond2 l(0/1).y s n yr3-yr26, gmm(s l.y) iv(n yr3-yr26) noleveleq r
```

### 연습 4.23

```stata
* continue
xtdpdsys y n yr3-yr26, pre(s) vce(r)
```

### 연습 4.24

```stata
use growth-ex, clear
by id: egen lny0 = total(lny / (year==0)), missing
```

## 4.5 임의효과 동태패널 모형

### 연습 4.25

```stata
use growth-ex, clear
by id: egen lny0 = total(lny / (year==0)), missing
forvalue k = 1/10 {
  by id: egen x1_`k' = total(x1 / (year==`k')), missing
}
xtreg lny x1 lny1 x1_1-x1_10 lny0, mle
```

## 4.6 특수한 경우

### 연습 4.26

```stata
use growth-ex, clear
qui tab year, gen(yr)
xtdpdsys lny x1 yr3-yr11, pre(x2) endo(x3) two vce(r)
```

### 연습 4.27

```stata
use growth-ex, clear
qui tab year, gen(yr)
xtdpdsys lny x1 yr2-yr11, pre(x2) endo(x3) two vce(r)
xtdpdsys lny x1 yr4-yr11, pre(x2) endo(x3) two vce(r)
```

### 연습 4.29

```stata
use growth-ex, clear
qui tab year, gen(yr)
xtabond lny x1 yr3-yr11, pre(x2) endo(x3) vce(r)
xtdpd l(0/1).lny x1 x2 x3 yr3-yr11, dgmm(x2, lag(1 .)) dgmm(lny x3) ///
  div(x1 yr3-yr11) hascons vce(r)
```

### 연습 4.30

```stata
use growth-ex, clear
qui tab year, gen(yr)
xtdpdsys lny x1 yr3-yr11, pre(x2) endo(x3) vce(r)
xtdpd l(0/1).lny x1 x2 x3 yr3-yr11, dgmm(x2, lag(1 .)) dgmm(lny x3) ///
   div(x1 yr3-yr11) lgmm(x2, lag(0)) lgmm(lny x3) hascons vce(r)
```

### 연습 4.31

```stata
use growth-ex, clear
qui tab year, gen(yr)
xtdpdsys lny x1 yr3-yr11, pre(x2) endo(x3) two vce(r)
xtdpd growth l.lny x1 x2 x3 yr3-yr11, dgmm(x2, lag(1 .)) dgmm(lny x3) ///
   div(x1 yr3-yr11) lgmm(x2, lag(0)) lgmm(lny x3) hascons vce(r) two
```

### 연습 4.32

```stata
use growth-ex, clear
qui tab year, gen(yr)
xtabond lny x1 yr3-yr11, pre(x2) endo(x3) vce(r)
xtabond2 l(0/1).lny x1 x2 x3 yr3-yr11, gmm(x2 l.(lny x3)) ///
   iv(x1 yr3-yr11) noleveleq r
```

### 연습 4.33

```stata
use growth-ex, clear
qui tab year, gen(yr)
xtdpdsys lny x1 yr3-yr11, pre(x2) endo(x3) vce(r)
xtabond2 l(0/1).lny x1 x2 x3 yr3-yr11, gmm(x2 l.(lny x3)) ///
   iv(x1 yr3-yr11, eq(d)) h(2) r
```

### 연습 4.34

```stata
use growth-ex, clear
qui tab year, gen(yr)
xtabond2 growth l.lny x1 x2 x3 yr3-yr11, gmm(x2 l.(lny x3)) ///
   iv(x1 yr3-yr11, eq(d)) h(2) r two
```

# Part II 비선형 패널 모형

# 5. 모집단평균 모형과 개체별 모형

### 연습 5.1

```stata
use union, clear
global model "union age grade not_smsa south black i.year"
reg ${model}, vce(cl idcode)
xtreg ${model}, pa corr(ind) vce(r)
xtreg ${model}, re vce(r)
xtreg ${model}, pa corr(exc) vce(r)
```

### 연습 5.2

```stata
use union, clear
global model "union age grade not_smsa south black i.year"
probit ${model}, vce(cl idcode)
xtprobit ${model}, pa corr(ind) vce(r)
xtprobit ${model}, re
xtprobit ${model}, pa corr(exc)
```

# 6. 이항반응모형

### 연습 6.1

```stata
use lfp, clear
d
xtset id period
xtsum lfp kids lhinc educ black age agesq

global z "educ black age agesq"
reg lfp kids lhinc ${z} i.period, vce(cl id)
xtreg lfp kids lhinc i.period, fe vce(r)

probit lfp kids lhinc ${z} i.period, vce(cl id)
xtprobit lfp kids lhinc ${z} i.period, pa corr(ind)
```

### 연습 6.2

계속

```stata
xtprobit lfp kids lhinc ${z} i.period, pa corr(exc)
```

### 연습 6.3

계속

```stata
xtprobit lfp kids lhinc ${z} i.period, pa corr(exc) vce(r)
```

계속

### 연습 6.4

```stata
xtprobit lfp kids lhinc ${z} i.period, re
```

### 연습 6.5

```stata
use union, clear
xtsum
global model "union age grade not_smsa south black i.year"
xtprobit ${model}, re
foreach v of varlist age grade not_smsa south {
  by id: egen `v'_bar = mean(`v')
}
xtprobit ${model} *_bar, re

drop not_smsa_*
tab year
forv yr = 70/88 {
  by id: egen not_smsa_`yr' = total(not_smsa / (year==`yr')), missing
}
xtprobit ${model} *_bar not_smsa_*, re

foreach v of varlist not_smsa_* {
  su `v', meanonly
  if `r(N)' == 0 {
    drop `v'
  }
}

xtprobit ${model} *_bar not_smsa_*, re
```

### 연습 6.6

```stata
use lfp, clear
xtset id period
global z "educ black age agesq"

by id: egen kidsbar = mean(kids)
by id: egen lhincbar = mean(lhinc)
xtprobit lfp kids lhinc kidsbar lhincbar ${z} i.period, re
```

### 연습 6.7

```stata
use vv98, clear
forv yr = 1981/1987 {
  by nr: egen mar_`yr' = total(mar / (year==`yr')), missing
}
by nr: egen union80 = total(union / (year==1980)), missing
xtprobit union mar l.union mar_* union80 school black i.year, re
// coef on mar = .168908
```

### 연습 6.8

```stata
use lfp, clear
xtset id period

forv t = 2/5 {
  foreach v of varlist kids lhinc {
    by id: egen `v'_`t' = total(`v' / (period==`t')), missing
  }
}
by id: egen lfp1 = total(lfp / (period==1)), missing

xtprobit lfp l.lfp kids lhinc educ black age agesq lfp1 kids_* lhinc_* i.period, re
// coef of l.lfp = 1.543212
```

### 연습 6.9

```stata
use union, clear
foreach v of varlist age grade not_smsa south {
  by id: egen `v'_bar = mean(`v')
}

global model "union age grade not_smsa south black i.year"

xtlogit $model, re nolog
est store re
xtlogit $model *_bar, re nolog
est store cre
xtlogit $model, fe nolog
est store fe
est tab re cre fe, b se stats(N)
```
