
# psidR: make building panel data from PSID easy

This R package provides a function to easily build panel data from PSID raw data.

### PSID

The [Panel Study of Income Dynamics](http://psidonline.isr.umich.edu/) is a publicly available dataset. You have to register and agree to terms and conditions, but there are no other strings attached. 

* you can use the [data center](http://simba.isr.umich.edu/default.aspx) to build simple datasets
* not workable for larger datasets
  * some variables don't show up (although you know they exist)
  * the ftp interface gets slower the more periods you are looking at
  * the click and scroll exercise of selecting the right variables in each period is extremely error prone. 
* merging the data manually is tricky.

### psidR

this package attempts to help the task of building a panel data. the user can either

1. download ASCII data from the server to disk and process with Stata or SAS to generate .dta or .csv files as input; or
2. there is an option to directly download into an R data.frame via SAScii (**caution** it takes long: the individual index has about 280MB in ASCII)

To build the panel, the user must specify the variable names in each wave of the questionnaire in a data.frame `fam.vars`, as well as the variables from the individual index in `ind.vars`. This will in almost all cases contain the **survey weights** you want to use. 

Please check out [the R survey package](http://cran.r-project.org/web/packages/survey/index.html) for analyzing complex survey's with R. Also go to [http://www.asdfree.com/](http://www.asdfree.com/) for a range of tutorials and tips for using survey data with R.


There are several prelimiary steps you have to take before using **psidR**. They all have to do with acquiring the data and storing it in a certain format. I'll explain below in examples.

#### In case you go for psidR option 1 

* download the zipped family data from [http://simba.isr.umich.edu/Zips/ZipMain.aspx](http://simba.isr.umich.edu/Zips/ZipMain.aspx)
  * run any of the contained program statements in each of the downloaded folders
* download the cross-year individual file
* the user can set some sample design options
* subsetting criteria
* if some variables are not measured in a given wave for whatever reason, the package takes care of that (after you tell it which ones are missing. see examples in package).

#### If you go for psidR option 2

You don't have to prepare anything: just enough time (you should think about leaving your machine on over night/the weekend, depending on how many waves you want to use. The individual index file is very big).

### How to install this package

The package is on CRAN, so just type

```r
install.packages('psidR')
```

Alternatively to get the development version from this repository,

```r
install.packages('devtools')
install_github("psidR",username="floswald")
```


### Example Usage

the main function in the package is `build.panel` and it has a reproducible example which you can look at by typing

```r
require(psidR)
example(build.panel)
```

#### Usage Outline

Suppose the user wants to have a panel with variables "house value", "total income" and "education" covering years 2001 and 2003. Steps 1 and 2 are relevant only for **option 1**, **option 2** requires only step 3 and 4:

1. Download the zipped family files and cross-period individual files from [http://simba.isr.umich.edu/Zips/ZipMain.aspx](http://simba.isr.umich.edu/Zips/ZipMain.aspx), best into the same folder. This folder will be the function argument `datadir`.
2. inside each downloaded folder, run the stata, sas or spss routine that comes with it. Fixes the text file up into a rectangular dataset. Save the data as either .dta or .csv. The default of the package requires that you use file names **FAMyyyy.dta** and **IND2009ER.dta** (case sensitive). 
3. Supply a data.frame **fam.vars** which contains the variable names for each wave from the family file.
4. Supply a data.frame **ind.vars** which contains the variable names for each wave from the individual index file.

```r
myvars <- data.frame(year=c(2001,2003),
                       house.value=c("ER17044","ER21043"),
                       total.income=c("ER20456","ER24099"),
                       education=c("ER20457","ER24148"))
indvars1 = data.frame(year=c(2001,2003),longitud.wgt=c("ER33637","ER33740"))
```

5. call the function, with `SAScii=TRUE` or `SAScii=FALSE` depending on your choice:

```r
option.1 <- build.panel(datadir=mydir,fam.vars=myvars,ind.vars=indvars,SAScii=FALSE)
option.2 <- build.panel(datadir=mydir,fam.vars=myvars,ind.vars=indvars,SAScii=TRUE)
```


Stata users may recognize this syntax from module [psiduse](http://ideas.repec.org/c/boc/bocode/s457040.html), which is similar. The names are up to you ("house.value" is your choice), but the rest is not, i.e. there must be a column "year". Notice if you knew house.value was missing in year 2001, you could account for that with 

```r
fam.vars <- data.frame(year=c(2001,2003),
                       house.value=c(NA,"ER21043"),
                       total.income=c("ER20456","ER24099"),
                       education=c("ER20457","ER24148"))
```

The function will then keep NA as the value of the variable in year 2001 and you can fix this later on. This functionality was needed because NAs have a generic meaning, i.e. a person who does not participate in a given year is kept in the register, but has no replies in the family file, so has NA in all variables of the family file after merging.

3. Specify options for the panel, like *design* or *heads.only*
4. call the function **build.panel**
5. the result is a wide data.table where the id colums are *pid* (person identifier) and *year*. 

### Supplemental Datasets

The PSID has a wealth of add-on datasets. Once you have a panel those are easy to merge on. The panel will have a variable `interview`, which is the identifier in the supplemental dataset. Just subset panel to the relevant year and do something like that (not tested):

```r
# suppose d is the result of an example call from above
# suppose variable V is in supplement X2001 and X2003 as a data.table
panel <- d$data	 	# get the data
setkey(d,year,interview) 	# set key
X2001[,year := 2001]	# add year column
X2003[,year := 2003]
setkey(X2001,year,interview)
setkey(X2003,year,interview)

# join X2001 to d (i.e. "merge")
d <- copy( d[X2001] )	# should add NA for year 2003
d <- copy( d[X2003] )
```

### Future Developments

* allow more complex panel designs, like accounting for wider family structure (i.e. using the family splitoff indicator to follow households that split up).


