자료를 다운로드 받지 않고 사용하려면 Stata에서 예를 들어 다음과 같이
명령함(마지막 `testfe`를 적절한 파일로 바꿈)

```stata
global datadir "https://github.com/chan079/Panel-Tutorial-BOK/raw/main/data"
use $datadir/testfe, clear
```

# 1. 도입

## 1.1 Stata 연습

### 연습 1.4

```stata
set more off
cd "c:/Documents/Data Folder"
log close _all
log using mylog.smcl, replace
*** Work here ***
log close
translate mylog.smcl mylog.pdf
set more on
```

## 1.2 계량경제학의 기초

### 연습 1.6

```stata
use death1, clear
reg deathrate smoke if year==2010
```

### 연습 1.7

아래에서 `*` 다음은 코멘트. `*` 대신에 `//`를 사용하면 `do` 파일에서는
괜찮지만 직접 명령창에 입력하면 오류 발생.

```stata
* continued
reg deathrate smoke aged if year==2010
```

### 연습 1.8

```stata
* continued
reg deathrate smoke aged i.year, vce(cl region)
```

### 연습 1.10

```stata
use hprice1, clear
reg lprice bdrms colonial
reg lprice bdrms colonial lsqrft
```

### 연습 1.11

```stata
gen lbdrmsize = ln(sqrft/bdrms)
reg lprice bdrms colonial lbdrmsize
```

### 연습 1.12

```stata
use mlb1, clear
reg lsalary years gamesyr bavg hrunsyr rbisyr
su years gamesyr bavg hrunsyr rbisyr
```

### 연습 1.13

```stata
use wage2, clear
reg lwage educ exper tenure married black south urban
```

### 연습 1.14

```stata
* continued
reg lwage educ exper tenure married black south urban IQ
```

### 연습 1.15 앞

```stata
use wage2, clear
reg hours lwage age married black
```

### 연습 1.18 앞

```stata
use death1, clear
reg deathrate drink smoke aged i.year
reg deathrate drink smoke aged i.year, vce(r)
reg deathrate drink smoke aged i.year, vce(cl region)
```

# Part I 선형패널모형

# 2. 선형패널모형과 추정

## 2.1 선형패널모형

## 2.2 집단 간 회귀, 통합 회귀, 집단 내 회귀

### 연습 2.2 앞

```stata
use wdi5bal, clear
xtreg sav age65over ggdppc i.year if oecd, be
```

### 연습 2.4 앞

```stata
use data, clear
xtreg sav age65over ggdppc i.year if oecd, be
```

### 연습 2.5 앞

```stata
use wdi5data, clear
reg sav age65over ggdppc i.year if oecd, vce(cl id)
```

### 연습 2.6 앞

```stata
use wdi5data, clear
xtreg sav age65over ggdppc i.year if oecd, fe vce(r)
```

### 연습 2.10 앞

```stata
use wdi5bal, clear
by id: egen bar_age65over = mean(age65over)
by id: egen bar_ggdppc = mean(ggdppc)
reg sav age65 gg i.year bar_*, vce(cl id)
xtreg sav age65 gg i.year, fe vce(r)
xtreg sav age65 gg, be
```

## 2.3 임의효과 모형과 고정효과 모형

### 연습 2.12 앞

```stata
use wdi5data, clear
xtreg sav age65over ggdppc i.year if oecd, re vce(r)
```

### 연습 2.17

```stata
use death1, clear
gen ldr = ln(deathrate)
gen lsmoke = ln(smoke)
gen laged = ln(aged)
global model ldr lsmoke laged i.year
xtreg $model, be
xtreg $model, fe vce(r)
reg $model, vce(cl region)
xtreg $model, re vce(r)
```

### 연습 2.18

```stata
use death1, clear
drop if year==2008
reg d.(deathrate smoke aged), nocons
reg d.(deathrate smoke aged)
xtreg deathrate smoke aged, fe
xtreg deathrate smoke aged i.year, fe
```

### 연습 2.19

```stata
* continued
reg d.(deathrate smoke aged) i.year
```

### 연습 2.20 앞

```stata
use wdi5data, clear
areg sav age65over ggdppc i.year if oecd, a(id) vce(cl id)
xtreg sav age65over ggdppc i.year if oecd, fe vce(r)
*ssc install reghdfe
reghdfe sav age65over ggdppc if oecd, a(id year) vce(cl id)
```

### 연습 2.21 앞

```stata
use testfe, clear
xtreg y x1 x2, fe
est store fe
xtreg y x1 x2 z1, re
hausman fe .
```

### 연습 2.22

```stata
use hausman-odd, clear
xtreg y x1 x2, fe
est store fe
xtreg y x1 x2 z1, re
hausman fe .
```

### 연습 2.23

```stata
use testfe, clear
xtreg y x1 x2, fe vce(r)
est store fe
xtreg y x1 x2, re vce(r)
est store re
hausman fe re
```

### 연습 2.24 앞

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

### 연습 2.29

```stata
use ict, clear
d
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
xtdes
tab sector if year==2010
xtreg lsales lcap lemp foreign i.year, fe
est store fe
xtreg lsales lcap lemp foreign kospi i.sector i.year, re
est store re
hausman fe re, sig
hausman fe re, sigmal
hausman fe re
xtreg lsales lcap lemp foreign kospi i.sector, be
foreach v of varlist lemp lcap foreign {
  by id: egen `v'_bar = mean(`v')
}
reg lsales lcap lemp foreign kospi i.sector i.year *_bar, vce(cl id)
testparm *_bar
```

## 2.4 Population-Averaged 모형

### 연습 2.30 앞

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

### 연습 2.31 앞

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

### 연습 2.36

```stata
use death1, clear
reg deathrate smoke
est store pols
xtreg deathrate smoke, be
est store be
xtreg deathrate smoke, re
est store re
xtreg deathrate smoke, fe
est store fe
areg deathrate smoke, a(region)
est store lsdv
est tab be pols re fe lsdv, stat(r2 r2_w r2_b r2_o)
```

### 2.5.2절 시작 부분

```stata
use fastfood, clear
xtset id after
gen d = nj & after
keep if balanced
xtreg fte d, fe vce(r)
xtreg fte d i.after, fe vce(r)
```

### 연습 2.38 앞

```stata
* continue
reg fte i.nj##i.after, vce(cl id)
```

### 연습 2.40

```stata
use fastfood, clear
xtset id after
gen d = nj & after
reg fte d i.nj i.after, vce(cl id)
xtreg fte d i.after, fe vce(r)
```

### 연습 2.41

```stata
* continue
reg fte d i.nj i.after, vce(cl id)
xtreg fte d i.after, fe vce(r)
```

### 연습 2.45 다음

```stata
use did3ex, clear
xtset
gen d = tgroup & after
table period tgroup, statistic(sum d)
xtreg y d i.period if inlist(period,0,1), fe vce(r)
xtreg y d i.period if inlist(period,0,2), fe vce(r)
di (.4548742+.6403278)/2
xtreg y d i.period, fe vce(r)
```

### 연습 2.46

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

### 2.5.3절

```stata
use https://friosavila.github.io/playingwithstata/drdid/mpdta.dta, clear
table year first
xtset county year
gen d = treat & year >= first_treat
xtdidregress (lemp) (d), group(countyreal) time(year)
estat bdecomp

* csdid
*ssc install csdid
csdid lemp, ivar(countyreal) time(year) gvar(first_treat)
estat simple
estat calendar
estat event

* jwdid
*ssc install jwdid
jwdid lemp, ivar(countyreal) tvar(year) gvar(first_treat) group never
estat event
estat plot

* did_imputation
* Data from https://ars.els-cdn.com/content/image/1-s2.0-S0014292123002210-mmc1.zip
* ./replication/data/monthly-panel.dta
use bjs-monthly-panel, clear
*ssc install did_imputation
did_imputation Y city_id month_stat month_intro, horizons(0/18) pretrend(12) avgeffectsby(cohorts_half_year) leaveout
event_plot, default_look
```

### 연습 2.47

[여기 참조](https://github.com/chan079/panelbook/blob/main/codes/ch04/ex04-09.do)

### 연습 2.48

[여기 참조](https://github.com/chan079/panelbook/blob/main/codes/ch04/ex04-10.do)

### 연습 2.50

```stata
*ssc install sdid
use synth_smoking, clear
gen treated = state==3 & year>=1989
sdid cigsale state year treated, vce(placebo) seed(1)
```

### 2.5.6절

아래에서 줄 마지막의 `///`는 ‘다음 줄로 계속됨’을 뜻한다. `do`
파일 내에서는 작동하나 직접 명령창에 입력하면 오류가 발생할 것이다.
직접 명령창에 입력하려면 `///` 없이 한 줄에 입력해야 한다.

```stata
use psidextract, clear
xthtaylor lwage wks south smsa ms exp exp2 occ ind union fem blk ed, ///
   endog(exp exp2 occ ind union ed)
```

# 3. 강외생적인 도구변수를 이용한 추정

## 3.1 내생성

## 3.2 강외생적 도구변수를 이용한 추정

### 연습 3.1

[여기 참조](https://github.com/chan079/panelbook/blob/main/codes/ch05/ex05-03.do)

## 3.3 Bartik Instruments

# 4. 선형 동적 패널 모형의 추정

## 4.1 선형 동적 패널 모형

## 4.2 고정효과 동태적 패널 모형의 GMM 추정

### 연습 4.4 다음, 연습 4.5 앞

```stata
use ajry08five, clear
xtabond dem yr3-yr11 if sample==1, pre(inc_1) vce(r) nocons
```

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

### 연습 4.20

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

### 연습 4.21

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

## 4.5 임의효과 동태패널 모형

### 연습 4.23

```stata
use growth-ex, clear
tab year
by id: egen lny_0 = total(lny / (year==0)), missing
* see data browser
```

### 연습 4.24

```stata
use growth-ex, clear
tab year
by id: egen lny_0 = total(lny / (year==0)), missing
forv j=1/10 {
  by id: egen x1_`j' = total(x1 / (year==`j')), missing
}
xtreg lny x1 l.lny i.year lny_0 x1_*, mle
```

## 4.6 특수한 경우

### 연습 4.25

```stata
use growth-ex, clear
qui tab year, gen(yr)
xtdpdsys lny x1 yr3-yr11, pre(x2) endo(x3) two vce(r)
```

### 연습 4.26

```stata
use growth-ex, clear
qui tab year, gen(yr)
xtdpdsys lny x1 yr2-yr11, pre(x2) endo(x3) two vce(r)
xtdpdsys lny x1 yr4-yr11, pre(x2) endo(x3) two vce(r)
```

### 연습 4.28

```stata
use growth-ex, clear
qui tab year, gen(yr)
xtabond lny x1 yr3-yr11, pre(x2) endo(x3) vce(r)
xtdpd l(0/1).lny x1 x2 x3 yr3-yr11, dgmm(x2, lag(1 .)) dgmm(lny x3) ///
  div(x1 yr3-yr11) hascons vce(r)
```

### 연습 4.29

```stata
use growth-ex, clear
qui tab year, gen(yr)
xtdpdsys lny x1 yr3-yr11, pre(x2) endo(x3) vce(r)
xtdpd l(0/1).lny x1 x2 x3 yr3-yr11, dgmm(x2, lag(1 .)) dgmm(lny x3) ///
   div(x1 yr3-yr11) lgmm(x2, lag(0)) lgmm(lny x3) hascons vce(r)
```

### 연습 4.30

```stata
use growth-ex, clear
qui tab year, gen(yr)
xtdpdsys lny x1 yr3-yr11, pre(x2) endo(x3) two vce(r)
xtdpd growth l.lny x1 x2 x3 yr3-yr11, dgmm(x2, lag(1 .)) dgmm(lny x3) ///
   div(x1 yr3-yr11) lgmm(x2, lag(0)) lgmm(lny x3) hascons vce(r) two
```

### 연습 4.31

```stata
use growth-ex, clear
qui tab year, gen(yr)
xtabond lny x1 yr3-yr11, pre(x2) endo(x3) vce(r)
xtabond2 l(0/1).lny x1 x2 x3 yr3-yr11, gmm(x2 l.(lny x3)) ///
   iv(x1 yr3-yr11) noleveleq r
```

### 연습 4.32

```stata
use growth-ex, clear
qui tab year, gen(yr)
xtdpdsys lny x1 yr3-yr11, pre(x2) endo(x3) vce(r)
xtabond2 l(0/1).lny x1 x2 x3 yr3-yr11, gmm(x2 l.(lny x3)) ///
   iv(x1 yr3-yr11, eq(d)) h(2) r
```

### 연습 4.33

```stata
use growth-ex, clear
tab year, gen(yr)
xtabond2 growth l.lny x1 x2 x3 yr3-yr11, gmm(x2 l.(lny x3)) iv(x1 yr3-yr11, eq(d)) h(2) two r
* rhohat = -.511922
```

# Part II 비선형 패널 모형

# 5. 이항반응모형

## 5.1 PA 모형

### 연습 5.1

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

### 연습 5.2

```stata
* continue
xtprobit lfp kids lhinc ${z} i.period, pa corr(exc)
```

### 연습 5.3

```stata
* continue
xtprobit lfp kids lhinc ${z} i.period, pa corr(ind) vce(r)
xtprobit lfp kids lhinc ${z} i.period, pa corr(exc) vce(r)
```

## 5.2 RE 모형

### 연습 5.4

```stata
use lfp, clear
global z "educ black age agesq"
xtprobit lfp kids lhinc ${z} i.period, re
est store re
xtprobit lfp kids lhinc ${z} i.period, pa c(exc) vce(r)
est store pa
est tab pa re, eq(1)
```

### 연습 5.5

```stata
use lfp, clear
xtsum
global z "educ black age agesq"
foreach v of varlist kids lhinc {
    by id: egen bar_`v' = mean(`v')
}
xtprobit lfp kids lhinc ${z} bar_* i.period, re
```

## 5.4 동태적 패널 프로빗 모형

### 연습 5.6

```stata
use vv98, clear
xtset
tab year
by nr: egen union_1980 = total(union / (year==1980)), missing
forv k=1981/1987 {
  by nr: egen mar_`k' = total(mar / (year==`k')), missing
}
xtprobit union mar l.union union_1980 mar_* school black i.year, re
* coef on mar = .168908, L1.union = .8975104
```

### 연습 5.7

```stata
use lfp, clear
xtset
xtsum
by id: egen lfp1 = total(lfp/(period==1)), missing
foreach v of varlist kids lhinc {
  forv k=2/5 {
    by id: egen `v'_`k' = total(`v'/(period==`k')), missing
  }
}
xtprobit lfp l.lfp kids lhinc educ black age agesq lfp1 kids_* lhinc_* i.period, re
* coef on l.lfp = 1.543212, kids = -.1451527
```
