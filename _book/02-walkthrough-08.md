# Classroom Gradebook

## Introduction

Notes of this chapter references chapter 8 ([Walkthrough
2](https://datascienceineducation.com/c08.html): Approaching Gradebook
Data From a Data Science Perspective) of [Data Science in Education
Using R](https://datascienceineducation.com/).

The analysis centers around a common K-12 classroom tool: the gradebook.
Because this kind of data is on an individual student level this chapter
uses a simulated dataset. Data source is an Excel file named
`ExcelGradeBook.xlsx` generated from an Excel gradebook template,
[Assessment Types
Points](https://web.mit.edu/jabbott/www/excelgradetracker.html).


```r
# Load libraries
library(tidyverse)
#> ── Attaching packages ────────────────────────────────── tidyverse 1.3.1 ──
#> ✓ ggplot2 3.3.5     ✓ purrr   0.3.4
#> ✓ tibble  3.1.6     ✓ dplyr   1.0.7
#> ✓ tidyr   1.1.4     ✓ stringr 1.4.0
#> ✓ readr   2.1.1     ✓ forcats 0.5.1
#> ── Conflicts ───────────────────────────────────── tidyverse_conflicts() ──
#> x dplyr::filter() masks stats::filter()
#> x dplyr::lag()    masks stats::lag()
library(here)
#> here() starts at /Users/petzi/Documents/Meine-Repos/ldsr
library(readxl)
library(janitor)
#> 
#> Attaching package: 'janitor'
#> The following objects are masked from 'package:stats':
#> 
#>     chisq.test, fisher.test
library(dataedu)
```

## Transfer file

Some steps to import data are in the book described differently and IMHO
not completely. The book assumes that you have downloaded the book
repository. I will focus just on the transfer of the file itself.

To transfer the file to your computer there are two options:

-   You could go manually to the location on the internet. The file is
    available at the [book
    repository](https://github.com/data-edu/data-science-in-education)
    inside the folder "gradebook" which itself is under the folder
    "data". You could go to the repo page
    (<https://github.com/data-edu/data-science-in-education/blob/master/data/gradebooks/ExcelGradeBook.xlsx>)
    and click on the "Download" button to save the file on your hard
    disc in the prepared RStudio project folder.
-   Our --- my preferred solution --- you could download the file
    directly via R. In this case we need to solve a general problem: How
    can you make sure that your colleagues are able to use your code?
    They have on their computers a different file organisation, so that
    using an absolute file path would not work. Even a relative path is
    dependent from the folder you are where you are going to start the
    procedure.

### Using `here()`

Providing file locations for loading or saving files is under the
reproducibility perspective a big challenge. It is also a general
problem so that it pays the effort to develop a special package just to
solve this problem. The general solution is implemented in the
**{rprojroot}** package. [rprojroot](https://rprojroot.r-lib.org/) helps
accessing files relative to the project root to [stop the working
directory
insanity](https://gist.github.com/jennybc/362f52446fe1ebc4c49f). Nut we
will use the **{here}** package, which is derived from **{rprojroot}**
but much simpler to use for our purpose.

The `here()` function from the **{here}** package uses a reasonable
heuristic to locate your file relative to the project root. This means
that you only have to add the file path from your working directory.
There are two modes to do this:

-   You write one complete expression like
    `here("some/path/below/your/project/root.txt")` or
-   you write every part separately like
    `here("some", "path", "below", "your", "project", "root.txt")`

I prefer the second option with a somewhat different layout, so that I
can see more clearly the directory hierarchy:

`here("some",        "path",        "below",        "your",              "project",        "root.txt"      )`

Before you can transfer the file to your computer you need to prepare a
place where you would like to store the file. I recommend to use the
same path hierarchy as in the book repository: Create a folder
`gradebook` inside of a folder `data` that itself lives at the top level
in your project directory.

You could create the folders manually either on the level of my
operation system or via the file pane of RStudio. The advantage is that
I do not have to learn and apply R commands for this procedure. The
disadvantage is that the code is not full reproducible. If you want
secure that your colleagues are working with exact the same environment
than you, than provide the necessary code, but commented out. Why
commented out? Nobody wants that a program changes something on the
private local hard disk without the explicit permission of the owner.

Now let's apply `here()` and transfer the file to your computer:

First of all we need to prepare the file path and create the folder
structure. We need to do this from a standard reference point, e.g.,
from the perspective of the project directory by using the `here()`
function.

To get the download URL we need to right-click at the "Download" button
at the location reported above and to copy the internet address into the
clipboard. To ease the handling of this long string we are going to
store it under the variable name `gradebook_url`. Then we will use the
`download.file()`, which is part of every R installation.

`download.file()` has several options, but we are going to use only the
two mandatory parameters: The URL where we will get the file and the
local path where we will store the transferred file. Again we will use
for the file path the `here()` function.


```r
## comment out the next line if you want create the folder structure programmatically
# dir.create(here("data", "folder"), recursive = TRUE)

## store download link into a variable
gradebook_url <- 
    "https://github.com/data-edu/data-science-in-education/raw/master/data/gradebooks/ExcelGradeBook.xlsx"

## download and store the file
download.file(gradebook_url, here("data", 
                                  "gradebooks", 
                                  "ExcelGradeBook.xlsx"))
```

If you have succeeded, you should see the following text under the R chunk:

::: greybox
trying URL '<https://github.com/data-edu/data-science-in-><br>
education/raw/master/data/gradebooks/ExcelGradeBook.xlsx'<br> Content
type 'application/octet-stream' length 116083 bytes (113 KB)<br>
==================================================<br> downloaded 113
KB<br>
:::

To secure that you do not always download the same file when you run
your program code I recommend to comment out these lines. Or --- if you
are using RMarkdown like me --- write the option "eval=FALSE" in the
chunk header. (This was the reason that you could not see the original
message in the R chunk above so that I had to reproduce the message
myself.)

### Import file into R

You have now the Excel file `ExcelGradeBook.xlsx` on our local hard disk.
The next step is to import it into R.

The recommended file format for working with dataset in R is the `.csv`
(comma-separated-value) format: These files are plain text and not a
proprietary format. To import `.csv` or other tabular data (tsv =
tabulator separated files, fwf = fixed-width files or web log files) one
uses the **{readr}** package, which is part or the tidyverse.

Nowadays there is with **{vroom}** another implementation of delimited
and fixed data to R. [Vroom](https://vroom.r-lib.org/) is 25 times
faster than **{readr}**, has almost all the parsing feature of
**{readr}** and even other new features as well. Until today I haven't
seen many real life application with **{vroom}**: Either it is still too
early or there are other consideration I do not know to keep using
**{readr}**. One reason might that the speed advantages of **{vroom}**
count only with very big datasets. The performance benchmark test for
example was done with a 1.55 GB dataset. Following this assumption
**{readr}** is still the predominant file package for reading in
delimited and fixed data to R.

However, data won't always come in the preferred `.csv` file format.
This walkthrough imports an Excel file because these file types, with
the `.xlsx` or `.xls` extensions, are very likely to be encountered in
the K-12 education world.

The book code uses the `read_excel()` function of the **{readxl}**
package to read the data of the locally stored file
(`ExcelGradeBook.xlsx`). **{readxl}** is part of the tidyverse and
supplements **{readr}**, the main tidyverse program for data import. The
`read_excel()` functions determines if the format is the legacy `.xls`
or the modern xml-based `.xlsx` format. Because of the file extension we
know that `ExcelGradeBook.xlsx` is in `.xlsx` format, we could also use
the slightly faster `read_xlsx()`.

::: infobox
Use the **{readODS}** package for reading and writing OpenDocument
Spreedsheets (ODS files). ODS files are derived from the Open Document
Format (ODF), an OpenSource ISO-certifacted standard for office
applications.

OpenSource software like [LibreOffice](https://www.libreoffice.org/)
uses this standard. It is the OpenSource alternative for the proprietary
Microsoft file format.
:::

### Inspect file

One necessary step during the import procedure is to inspect the file. The
inspection is necessary to know about the structure of the dataset. There are at least
three possible ways to do this:

1.  Using R code --- as the book demonstrates --- to inspect the file.
2.  Using appropriate software to open the file and inspect it.
3.  Using RStudio interactive dataset importing tools under the menu
    "File" -> "Import dataset" -> "From Excel...".
    
For demonstrative purposes we are going to apply all three options.

#### Using R code

We will use for the local file path again here() and load "ExcelGradeBook.xlsx" into the R object `ExcelGradeBook`. As the book suggests we will make a copy with a better name (`gradebook`), that is easier to remember and type. We will later on work with this copy and have leave the original as a backup if anything goes wrong.

#### Standard printing

Then we display the content of `gradebook` to inspect its content.


```r
# Use readxl package to read and import file and assign it a name
ExcelGradeBook <-
  read_excel(
    here("data", 
         "gradebooks", 
         "ExcelGradeBook.xlsx")
  )
#> New names:
#> * `` -> ...1
#> * `` -> ...3
#> * `` -> ...4
#> * `` -> ...5
#> * `` -> ...6
#> * ...

# Copy R object to have a working file and a backup
gradebook <- ExcelGradeBook 
gradebook
#> # A tibble: 43 × 63
#>    ...1  `Assessment Type` ...3  ...4   ...5  ...6  ...7  ...8  ...9  ...10
#>    <chr> <chr>             <chr> <chr>  <chr> <chr> <chr> <chr> <chr> <chr>
#>  1 <NA>  Points            <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#>  2 <NA>  Point Multiplier  <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#>  3 <NA>  Date              <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#>  4 <NA>  <NA>              <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#>  5 A-    90                <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#>  6 B-    80                <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#>  7 C-    70                <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#>  8 D-    60                <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#>  9 F:    0                 <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#> 10 Class Name              Race  Gender Age   Repe… Fina… Abse… Late  Make…
#> # … with 33 more rows, and 53 more variables: ...11 <chr>, ...12 <chr>,
#> #   ...13 <chr>, ...14 <chr>, ...15 <chr>, ...16 <chr>, ...17 <chr>,
#> #   ...18 <chr>, Classworks...19 <chr>, Homeworks...20 <chr>,
#> #   Classworks...21 <chr>, Homeworks...22 <chr>, Classworks...23 <chr>,
#> #   Classworks...24 <chr>, Classworks...25 <chr>, Classworks...26 <chr>,
#> #   Homeworks...27 <chr>, Formative Assessments...28 <chr>,
#> #   Projects...29 <chr>, Classworks...30 <chr>, Homeworks...31 <chr>, …
```

Oops! What has happened here? If you inspect the content of the dataset
you will learn that

-   we've got a dataset with 43 columns and 63 rows
-   without sensible columns names (R has warned us that it created many columns automatically)
-   with 25 students under the heading of `Assessment Type`
-   with many NA rows at the beginning but also at the end of the
    dataset.

The reason for this strange structure is that someone has used the first
9 lines of the Excel file for a note about the grading system. As
important as this note may be, it runs counter the R assumption that the
first line will be a row with the column names.

#### Using `print()`

Unfortunately the standard printing procedure of R with tibble datasets
(a special sort of data frames in the `tidyverse`) limits the
presentation of exact 10 rows per page. Therefore it is especially
difficult to see what has happened. It would be better to see the whole
dataset at once. This could be done with the `print()` command by stating explicitly how many rows are to print.


```r
### print file with specifies row numbers: for instance
## print(gradebook, n = 100)


### or specify the number with the `nrow()` function
## print(gradebook, n = nrow(gradebook))

### the same code but with pipe operator
gradebook %>% print(n = nrow(.)) 
#> # A tibble: 43 × 63
#>    ...1  `Assessment Type` ...3  ...4   ...5  ...6  ...7  ...8  ...9  ...10
#>    <chr> <chr>             <chr> <chr>  <chr> <chr> <chr> <chr> <chr> <chr>
#>  1 <NA>  Points            <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#>  2 <NA>  Point Multiplier  <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#>  3 <NA>  Date              <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#>  4 <NA>  <NA>              <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#>  5 A-    90                <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#>  6 B-    80                <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#>  7 C-    70                <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#>  8 D-    60                <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#>  9 F:    0                 <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#> 10 Class Name              Race  Gender Age   Repe… Fina… Abse… Late  Make…
#> 11 1     Student 1         <NA>  <NA>   <NA>  <NA>  <NA>  1     0     <NA> 
#> 12 1     Student 2         <NA>  <NA>   <NA>  <NA>  <NA>  0     1     <NA> 
#> 13 1     Student 3         <NA>  <NA>   <NA>  <NA>  <NA>  2     0     <NA> 
#> 14 1     Student 4         <NA>  <NA>   <NA>  <NA>  <NA>  0     0     <NA> 
#> 15 1     Student 5         <NA>  <NA>   <NA>  <NA>  <NA>  0     0     <NA> 
#> 16 1     Student 6         <NA>  <NA>   <NA>  <NA>  <NA>  0     0     <NA> 
#> 17 1     Student 7         <NA>  <NA>   <NA>  <NA>  <NA>  0     0     <NA> 
#> 18 1     Student 8         <NA>  <NA>   <NA>  <NA>  <NA>  0     0     <NA> 
#> 19 1     Student 9         <NA>  <NA>   <NA>  <NA>  <NA>  0     0     <NA> 
#> 20 1     Student 10        <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#> 21 1     Student 11        <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#> 22 1     Student 12        <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#> 23 1     Student 13        <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#> 24 1     Student 14        <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#> 25 1     Student 15        <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#> 26 1     Student 16        <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#> 27 1     Student 17        <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#> 28 1     Student 18        <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#> 29 1     Student 19        <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#> 30 1     Student 20        <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#> 31 1     Student 21        <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#> 32 1     Student 22        <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#> 33 1     Student 23        <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#> 34 1     Student 24        <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#> 35 1     Student 25        <NA>  <NA>   <NA>  <NA>  <NA>  No m… No m… <NA> 
#> 36 <NA>  <NA>              <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#> 37 <NA>  <NA>              <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#> 38 <NA>  <NA>              <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#> 39 <NA>  <NA>              <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#> 40 <NA>  <NA>              <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#> 41 <NA>  <NA>              <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#> 42 <NA>  <NA>              <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#> 43 <NA>  <NA>              <NA>  <NA>   <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
#> # … with 53 more variables: ...11 <chr>, ...12 <chr>, ...13 <chr>,
#> #   ...14 <chr>, ...15 <chr>, ...16 <chr>, ...17 <chr>, ...18 <chr>,
#> #   Classworks...19 <chr>, Homeworks...20 <chr>, Classworks...21 <chr>,
#> #   Homeworks...22 <chr>, Classworks...23 <chr>, Classworks...24 <chr>,
#> #   Classworks...25 <chr>, Classworks...26 <chr>, Homeworks...27 <chr>,
#> #   Formative Assessments...28 <chr>, Projects...29 <chr>,
#> #   Classworks...30 <chr>, Homeworks...31 <chr>, Projects...32 <chr>, …
```
#### Using `paged_table()`

Another option to inspect the whole dataset with R is:


```r
gradebook %>% rmarkdown::paged_table()
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["...1"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Assessment Type"],"name":[2],"type":["chr"],"align":["left"]},{"label":["...3"],"name":[3],"type":["chr"],"align":["left"]},{"label":["...4"],"name":[4],"type":["chr"],"align":["left"]},{"label":["...5"],"name":[5],"type":["chr"],"align":["left"]},{"label":["...6"],"name":[6],"type":["chr"],"align":["left"]},{"label":["...7"],"name":[7],"type":["chr"],"align":["left"]},{"label":["...8"],"name":[8],"type":["chr"],"align":["left"]},{"label":["...9"],"name":[9],"type":["chr"],"align":["left"]},{"label":["...10"],"name":[10],"type":["chr"],"align":["left"]},{"label":["...11"],"name":[11],"type":["chr"],"align":["left"]},{"label":["...12"],"name":[12],"type":["chr"],"align":["left"]},{"label":["...13"],"name":[13],"type":["chr"],"align":["left"]},{"label":["...14"],"name":[14],"type":["chr"],"align":["left"]},{"label":["...15"],"name":[15],"type":["chr"],"align":["left"]},{"label":["...16"],"name":[16],"type":["chr"],"align":["left"]},{"label":["...17"],"name":[17],"type":["chr"],"align":["left"]},{"label":["...18"],"name":[18],"type":["chr"],"align":["left"]},{"label":["Classworks...19"],"name":[19],"type":["chr"],"align":["left"]},{"label":["Homeworks...20"],"name":[20],"type":["chr"],"align":["left"]},{"label":["Classworks...21"],"name":[21],"type":["chr"],"align":["left"]},{"label":["Homeworks...22"],"name":[22],"type":["chr"],"align":["left"]},{"label":["Classworks...23"],"name":[23],"type":["chr"],"align":["left"]},{"label":["Classworks...24"],"name":[24],"type":["chr"],"align":["left"]},{"label":["Classworks...25"],"name":[25],"type":["chr"],"align":["left"]},{"label":["Classworks...26"],"name":[26],"type":["chr"],"align":["left"]},{"label":["Homeworks...27"],"name":[27],"type":["chr"],"align":["left"]},{"label":["Formative Assessments...28"],"name":[28],"type":["chr"],"align":["left"]},{"label":["Projects...29"],"name":[29],"type":["chr"],"align":["left"]},{"label":["Classworks...30"],"name":[30],"type":["chr"],"align":["left"]},{"label":["Homeworks...31"],"name":[31],"type":["chr"],"align":["left"]},{"label":["Projects...32"],"name":[32],"type":["chr"],"align":["left"]},{"label":["Classworks...33"],"name":[33],"type":["chr"],"align":["left"]},{"label":["Homeworks...34"],"name":[34],"type":["chr"],"align":["left"]},{"label":["Projects...35"],"name":[35],"type":["chr"],"align":["left"]},{"label":["Homeworks...36"],"name":[36],"type":["chr"],"align":["left"]},{"label":["Classworks...37"],"name":[37],"type":["chr"],"align":["left"]},{"label":["Homeworks...38"],"name":[38],"type":["chr"],"align":["left"]},{"label":["Homeworks...39"],"name":[39],"type":["chr"],"align":["left"]},{"label":["Projects...40"],"name":[40],"type":["chr"],"align":["left"]},{"label":["Projects...41"],"name":[41],"type":["chr"],"align":["left"]},{"label":["Formative Assessments...42"],"name":[42],"type":["chr"],"align":["left"]},{"label":["Projects...43"],"name":[43],"type":["chr"],"align":["left"]},{"label":["Classworks...44"],"name":[44],"type":["chr"],"align":["left"]},{"label":["Homeworks...45"],"name":[45],"type":["chr"],"align":["left"]},{"label":["Classworks...46"],"name":[46],"type":["chr"],"align":["left"]},{"label":["Homeworks...47"],"name":[47],"type":["chr"],"align":["left"]},{"label":["Classworks...48"],"name":[48],"type":["chr"],"align":["left"]},{"label":["Classworks...49"],"name":[49],"type":["chr"],"align":["left"]},{"label":["Projects...50"],"name":[50],"type":["chr"],"align":["left"]},{"label":["Classworks...51"],"name":[51],"type":["chr"],"align":["left"]},{"label":["Classworks...52"],"name":[52],"type":["chr"],"align":["left"]},{"label":["Homeworks...53"],"name":[53],"type":["chr"],"align":["left"]},{"label":["Formative Assessments...54"],"name":[54],"type":["chr"],"align":["left"]},{"label":["Classworks...55"],"name":[55],"type":["chr"],"align":["left"]},{"label":["Homeworks...56"],"name":[56],"type":["chr"],"align":["left"]},{"label":["Classworks...57"],"name":[57],"type":["chr"],"align":["left"]},{"label":["Homeworks...58"],"name":[58],"type":["chr"],"align":["left"]},{"label":["Projects...59"],"name":[59],"type":["chr"],"align":["left"]},{"label":["Projects...60"],"name":[60],"type":["chr"],"align":["left"]},{"label":["Projects...61"],"name":[61],"type":["chr"],"align":["left"]},{"label":["Summative Assessments"],"name":[62],"type":["chr"],"align":["left"]},{"label":["...63"],"name":[63],"type":["chr"],"align":["left"]}],"data":[{"1":"NA","2":"Points","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"NA","12":"NA","13":"Percentage Breakdown By Assessment Type","14":"NA","15":"NA","16":"NA","17":"NA","18":"NA","19":"15","20":"10","21":"15","22":"5","23":"15","24":"15","25":"15","26":"15","27":"10","28":"50","29":"10","30":"15","31":"10","32":"10","33":"15","34":"5","35":"10","36":"5","37":"15","38":"10","39":"10","40":"10","41":"10","42":"50","43":"10","44":"15","45":"10","46":"15","47":"5","48":"15","49":"15","50":"10","51":"15","52":"15","53":"10","54":"50","55":"15","56":"10","57":"15","58":"5","59":"10","60":"10","61":"10","62":"30","63":"NA"},{"1":"NA","2":"Point Multiplier","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"NA","12":"NA","13":"NA","14":"NA","15":"NA","16":"NA","17":"NA","18":"NA","19":"1","20":"1","21":"1","22":"1","23":"1","24":"1","25":"1","26":"1","27":"1","28":"1","29":"1","30":"1","31":"1","32":"1","33":"1","34":"1","35":"1","36":"1","37":"1","38":"1","39":"1","40":"1","41":"1","42":"1","43":"1","44":"1","45":"1","46":"1","47":"1","48":"1","49":"1","50":"1","51":"1","52":"1","53":"1","54":"1","55":"1","56":"1","57":"1","58":"1","59":"1","60":"1","61":"1","62":"1","63":"1"},{"1":"NA","2":"Date","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"NA","12":"NA","13":"0.15","14":"0.15","15":"0.15","16":"0.35","17":"0.2","18":"0","19":"43485","20":"43485","21":"43488","22":"43491","23":"43492","24":"43493","25":"43494","26":"43495","27":"43495","28":"43496","29":"43498","30":"43499","31":"43501","32":"43502","33":"43503","34":"43503","35":"43505","36":"43505","37":"43506","38":"43507","39":"43509","40":"43509","41":"43510","42":"43512","43":"43514","44":"43515","45":"43516","46":"43519","47":"43521","48":"43524","49":"43528","50":"43529","51":"43530","52":"43533","53":"43535","54":"43535","55":"43537","56":"43539","57":"43542","58":"43543","59":"43545","60":"43547","61":"43549","62":"43552","63":"NA"},{"1":"NA","2":"NA","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"NA","12":"NA","13":"NA","14":"NA","15":"NA","16":"NA","17":"NA","18":"NA","19":"NA","20":"NA","21":"NA","22":"NA","23":"NA","24":"NA","25":"NA","26":"NA","27":"NA","28":"NA","29":"NA","30":"NA","31":"NA","32":"NA","33":"NA","34":"NA","35":"NA","36":"NA","37":"NA","38":"NA","39":"NA","40":"NA","41":"NA","42":"NA","43":"NA","44":"NA","45":"NA","46":"NA","47":"NA","48":"NA","49":"NA","50":"NA","51":"NA","52":"NA","53":"NA","54":"NA","55":"NA","56":"NA","57":"NA","58":"NA","59":"NA","60":"NA","61":"NA","62":"NA","63":"NA"},{"1":"A-","2":"90","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"8","12":"NA","13":"5","14":"7","15":"8","16":"17","17":"12","18":"0","19":"6","20":"21","21":"NA","22":"13","23":"NA","24":"NA","25":"NA","26":"NA","27":"NA","28":"10","29":"20","30":"NA","31":"NA","32":"NA","33":"NA","34":"NA","35":"NA","36":"12","37":"9","38":"11","39":"15","40":"23","41":"17","42":"NA","43":"NA","44":"15","45":"11","46":"20","47":"9","48":"8","49":"13","50":"NA","51":"15","52":"17","53":"25","54":"13","55":"17","56":"17","57":"11","58":"8","59":"19","60":"17","61":"19","62":"12","63":"0"},{"1":"B-","2":"80","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"12","12":"NA","13":"5","14":"8","15":"10","16":"8","17":"7","18":"0","19":"4","20":"4","21":"NA","22":"11","23":"NA","24":"NA","25":"NA","26":"NA","27":"NA","28":"5","29":"1","30":"NA","31":"NA","32":"NA","33":"NA","34":"NA","35":"NA","36":"6","37":"3","38":"2","39":"1","40":"2","41":"8","42":"NA","43":"NA","44":"5","45":"0","46":"5","47":"1","48":"4","49":"7","50":"NA","51":"8","52":"7","53":"0","54":"8","55":"6","56":"1","57":"2","58":"3","59":"6","60":"3","61":"6","62":"7","63":"0"},{"1":"C-","2":"70","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"5","12":"NA","13":"7","14":"10","15":"5","16":"0","17":"5","18":"0","19":"5","20":"0","21":"NA","22":"0","23":"NA","24":"NA","25":"NA","26":"NA","27":"NA","28":"6","29":"4","30":"NA","31":"NA","32":"NA","33":"NA","34":"NA","35":"NA","36":"0","37":"2","38":"1","39":"2","40":"0","41":"0","42":"NA","43":"NA","44":"0","45":"0","46":"0","47":"0","48":"0","49":"1","50":"NA","51":"2","52":"0","53":"0","54":"4","55":"0","56":"2","57":"1","58":"0","59":"0","60":"5","61":"0","62":"5","63":"0"},{"1":"D-","2":"60","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"0","12":"NA","13":"7","14":"0","15":"2","16":"0","17":"0","18":"0","19":"3","20":"0","21":"NA","22":"0","23":"NA","24":"NA","25":"NA","26":"NA","27":"NA","28":"1","29":"0","30":"NA","31":"NA","32":"NA","33":"NA","34":"NA","35":"NA","36":"2","37":"3","38":"2","39":"2","40":"0","41":"0","42":"NA","43":"NA","44":"3","45":"1","46":"0","47":"1","48":"3","49":"4","50":"NA","51":"0","52":"1","53":"0","54":"0","55":"2","56":"2","57":"2","58":"7","59":"0","60":"0","61":"0","62":"0","63":"0"},{"1":"F:","2":"0","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"0","12":"NA","13":"1","14":"0","15":"0","16":"0","17":"1","18":"0","19":"7","20":"0","21":"NA","22":"0","23":"NA","24":"NA","25":"NA","26":"NA","27":"NA","28":"3","29":"0","30":"NA","31":"NA","32":"NA","33":"NA","34":"NA","35":"NA","36":"5","37":"8","38":"9","39":"5","40":"0","41":"0","42":"NA","43":"NA","44":"2","45":"13","46":"0","47":"14","48":"10","49":"0","50":"NA","51":"0","52":"0","53":"0","54":"0","55":"0","56":"3","57":"9","58":"7","59":"0","60":"0","61":"0","62":"1","63":"0"},{"1":"Class","2":"Name","3":"Race","4":"Gender","5":"Age","6":"Repeated Grades","7":"Financial Status","8":"Absent","9":"Late","10":"Make your own categories","11":"Running Average","12":"Letter Grade","13":"Homeworks","14":"Classworks","15":"Formative Assessments","16":"Projects","17":"Summative Assessments","18":"Another Type 2","19":"Classwork 1","20":"Homework 1","21":"Classwork 2","22":"Homework 2","23":"Classwork 3","24":"Classwork 4","25":"Classwork 5","26":"Classwork 6","27":"Homework 3","28":"Formative Assessment 1","29":"Project 1","30":"Classwork 7","31":"Homework 4","32":"Project 2","33":"Classwork 8","34":"Homework 5","35":"Project 3","36":"Homework 6","37":"Classwork 9","38":"Homework 7","39":"Homework 8","40":"Project 4","41":"Project 5","42":"Formative Assessment 2","43":"Project 6","44":"Classwork 10","45":"Homework 9","46":"Classwork 11","47":"Homework 10","48":"Classwork 12","49":"Classwork 13","50":"Project 7","51":"Classwork 14","52":"Classwork 15","53":"Homework 11","54":"Summative Assessment 1","55":"Classwork 16","56":"Homework 12","57":"Classwork 17","58":"Homework 13","59":"Project 8","60":"Project 9","61":"Project 10","62":"Summative Assessment 2","63":"Assessment | Insert new columns before here"},{"1":"1","2":"Student 1","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"1","9":"0","10":"NA","11":"99.3823529411765","12":"A+","13":"100","14":"99.2156862745098","15":"96.6666666666667","16":"100","17":"100","18":"NA","19":"13","20":"10","21":"15","22":"5","23":"15","24":"15","25":"15","26":"15","27":"10","28":"45","29":"10","30":"15","31":"10","32":"10","33":"15","34":"5","35":"10","36":"5","37":"15","38":"10","39":"10","40":"10","41":"10","42":"50","43":"10","44":"15","45":"10","46":"15","47":"5","48":"15","49":"15","50":"10","51":"15","52":"15","53":"10","54":"50","55":"15","56":"10","57":"15","58":"5","59":"10","60":"10","61":"10","62":"30","63":"NA"},{"1":"1","2":"Student 2","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"0","9":"1","10":"NA","11":"79.3519607843137","12":"C+","13":"53.3333333333333","14":"88.2352941176471","15":"86.6666666666667","16":"87","17":"73.3333333333333","18":"NA","19":"10","20":"9","21":"12","22":"4","23":"15","24":"12","25":"15","26":"15","27":"7","28":"44","29":"9","30":"11","31":"3","32":"10","33":"15","34":"0","35":"10","36":"4","37":"15","38":"0","39":"4","40":"9","41":"8","42":"46","43":"10","44":"12","45":"0","46":"15","47":"3","48":"6","49":"14","50":"5","51":"15","52":"15","53":"10","54":"40","55":"15","56":"9","57":"13","58":"3","59":"9","60":"7","61":"10","62":"22","63":"NA"},{"1":"1","2":"Student 3","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"2","9":"0","10":"NA","11":"86.65","12":"B+","13":"62","14":"88.3333333333333","15":"93.3333333333333","16":"86","17":"100","18":"NA","19":"10","20":"9","21":"12","22":"Excused","23":"Excused","24":"12","25":"15","26":"15","27":"7","28":"50","29":"9","30":"14","31":"3","32":"10","33":"15","34":"3","35":"10","36":"5","37":"15","38":"1","39":"4","40":"9","41":"8","42":"50","43":"8","44":"10","45":"3","46":"15","47":"4","48":"6","49":"14","50":"7","51":"14","52":"15","53":"10","54":"40","55":"15","56":"10","57":"15","58":"3","59":"9","60":"7","61":"9","62":"30","63":"NA"},{"1":"1","2":"Student 4","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"0","9":"0","10":"NA","11":"80.2235294117647","12":"B-","13":"60","14":"78.8235294117647","15":"84","16":"88","17":"80","18":"NA","19":"10","20":"9","21":"5","22":"4","23":"15","24":"12","25":"15","26":"15","27":"7","28":"40","29":"9","30":"13","31":"3","32":"10","33":"15","34":"2","35":"10","36":"4","37":"15","38":"3","39":"4","40":"10","41":"10","42":"46","43":"9","44":"9","45":"3","46":"15","47":"2","48":"6","49":"9","50":"5","51":"11","52":"15","53":"10","54":"40","55":"13","56":"9","57":"8","58":"3","59":"9","60":"8","61":"8","62":"24","63":"NA"},{"1":"1","2":"Student 5","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"0","9":"0","10":"NA","11":"86.5081232492997","12":"B+","13":"72.3809523809524","14":"74.1176470588235","15":"86.6666666666667","16":"92","17":"96.6666666666667","18":"NA","19":"7","20":"10","21":"11","22":"5","23":"8","24":"12","25":"7","26":"12","27":"10","28":"39","29":"10","30":"13","31":"5","32":"10","33":"15","34":"0","35":"10","36":"4","37":"14","38":"8","39":"9","40":"10","41":"10","42":"41","43":"10","44":"13","45":"3","46":"13","47":"1","48":"4","49":"13","50":"6","51":"15","52":"15","53":"9","54":"50","55":"15","56":"9","57":"2","58":"3","59":"8","60":"8","61":"10","62":"29","63":"NA"},{"1":"1","2":"Student 6","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"0","9":"0","10":"NA","11":"83.7872549019608","12":"B","13":"80","14":"76.4705882352941","15":"79.3333333333333","16":"85","17":"93.3333333333333","18":"NA","19":"14","20":"9","21":"8","22":"4","23":"6","24":"12","25":"7","26":"15","27":"10","28":"36","29":"10","30":"15","31":"9","32":"7","33":"15","34":"2","35":"9","36":"5","37":"13","38":"10","39":"10","40":"9","41":"9","42":"41","43":"9","44":"8","45":"1","46":"15","47":"1","48":"8","49":"12","50":"6","51":"11","52":"14","53":"9","54":"42","55":"13","56":"10","57":"9","58":"4","59":"8","60":"9","61":"9","62":"28","63":"NA"},{"1":"1","2":"Student 7","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"0","9":"0","10":"NA","11":"84.7556022408964","12":"B","13":"87.6190476190476","14":"76.8627450980392","15":"81.3333333333333","16":"93","17":"76.6666666666667","18":"NA","19":"11","20":"9","21":"11","22":"4","23":"13","24":"14","25":"14","26":"12","27":"10","28":"41","29":"10","30":"13","31":"10","32":"9","33":"15","34":"3","35":"10","36":"5","37":"9","38":"10","39":"10","40":"10","41":"9","42":"37","43":"10","44":"8","45":"10","46":"12","47":"1","48":"8","49":"12","50":"9","51":"12","52":"15","53":"10","54":"44","55":"15","56":"10","57":"2","58":"0","59":"9","60":"9","61":"8","62":"23","63":"NA"},{"1":"1","2":"Student 8","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"0","9":"0","10":"NA","11":"90.1767507002801","12":"A-","13":"85.7142857142857","14":"82.3529411764706","15":"89.3333333333333","16":"94","17":"93.3333333333333","18":"NA","19":"7","20":"10","21":"5","22":"4","23":"6","24":"14","25":"10","26":"8","27":"9","28":"41","29":"10","30":"15","31":"9","32":"9","33":"15","34":"1","35":"9","36":"4","37":"11","38":"10","39":"9","40":"10","41":"9","42":"43","43":"10","44":"15","45":"9","46":"15","47":"5","48":"15","49":"15","50":"10","51":"15","52":"15","53":"10","54":"50","55":"15","56":"9","57":"14","58":"1","59":"8","60":"9","61":"10","62":"28","63":"NA"},{"1":"1","2":"Student 9","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"0","9":"0","10":"NA","11":"92.2634453781513","12":"A-","13":"94.2857142857143","14":"83.1372549019608","15":"82.6666666666667","16":"95","17":"100","18":"NA","19":"5","20":"10","21":"5","22":"4","23":"14","24":"15","25":"10","26":"8","27":"10","28":"38","29":"8","30":"13","31":"9","32":"9","33":"15","34":"3","35":"10","36":"4","37":"8","38":"9","39":"10","40":"9","41":"10","42":"36","43":"9","44":"14","45":"10","46":"15","47":"5","48":"15","49":"15","50":"10","51":"15","52":"15","53":"10","54":"50","55":"15","56":"10","57":"15","58":"5","59":"10","60":"10","61":"10","62":"30","63":"NA"},{"1":"1","2":"Student 10","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"84.5361344537815","12":"B","13":"90.4761904761905","14":"71.7647058823529","15":"70.6666666666667","16":"96","17":"80","18":"NA","19":"8","20":"10","21":"6","22":"4","23":"15","24":"15","25":"9","26":"9","27":"10","28":"42","29":"10","30":"12","31":"10","32":"9","33":"15","34":"3","35":"10","36":"5","37":"8","38":"10","39":"10","40":"9","41":"10","42":"28","43":"10","44":"10","45":"10","46":"12","47":"0","48":"13","49":"9","50":"9","51":"12","52":"13","53":"10","54":"36","55":"15","56":"10","57":"2","58":"3","59":"10","60":"9","61":"10","62":"24","63":"NA"},{"1":"1","2":"Student 11","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"90.478431372549","12":"A-","13":"86.6666666666667","14":"96.078431372549","15":"94.6666666666667","16":"92","17":"83.3333333333333","18":"NA","19":"15","20":"9","21":"14","22":"5","23":"15","24":"15","25":"15","26":"15","27":"10","28":"50","29":"10","30":"15","31":"9","32":"10","33":"15","34":"5","35":"9","36":"0","37":"10","38":"9","39":"9","40":"10","41":"10","42":"37","43":"8","44":"15","45":"9","46":"14","47":"5","48":"15","49":"15","50":"10","51":"13","52":"14","53":"10","54":"55","55":"15","56":"9","57":"15","58":"2","59":"8","60":"9","61":"8","62":"25","63":"NA"},{"1":"1","2":"Student 12","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"83.7822128851541","12":"B","13":"83.8095238095238","14":"75.2941176470588","15":"84.6666666666667","16":"93","17":"73.3333333333333","18":"NA","19":"12","20":"9","21":"14","22":"5","23":"9","24":"15","25":"6","26":"6","27":"9","28":"35","29":"9","30":"11","31":"10","32":"10","33":"15","34":"2","35":"9","36":"1","37":"12","38":"10","39":"9","40":"10","41":"9","42":"47","43":"9","44":"15","45":"9","46":"15","47":"2","48":"8","49":"9","50":"9","51":"13","52":"13","53":"9","54":"45","55":"15","56":"9","57":"4","58":"4","59":"10","60":"10","61":"8","62":"22","63":"NA"},{"1":"1","2":"Student 13","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"95.5694677871149","12":"A","13":"97.1428571428571","14":"91.7647058823529","15":"100","16":"94","17":"96.6666666666667","18":"NA","19":"11","20":"9","21":"5","22":"5","23":"15","24":"13","25":"15","26":"15","27":"9","28":"50","29":"9","30":"15","31":"10","32":"10","33":"15","34":"5","35":"10","36":"5","37":"15","38":"9","39":"10","40":"9","41":"9","42":"50","43":"9","44":"15","45":"10","46":"14","47":"5","48":"15","49":"13","50":"9","51":"15","52":"13","53":"9","54":"50","55":"15","56":"10","57":"15","58":"6","59":"9","60":"10","61":"10","62":"29","63":"NA"},{"1":"1","2":"Student 14","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"78.7851540616247","12":"C+","13":"63.8095238095238","14":"70.9803921568628","15":"69.3333333333333","16":"90","17":"83.3333333333333","18":"NA","19":"5","20":"9","21":"7","22":"4","23":"13","24":"13","25":"5","26":"13","27":"9","28":"28","29":"7","30":"15","31":"9","32":"9","33":"15","34":"5","35":"9","36":"2","37":"5","38":"1","39":"4","40":"9","41":"8","42":"30","43":"10","44":"15","45":"6","46":"13","47":"2","48":"4","49":"10","50":"10","51":"12","52":"15","53":"10","54":"46","55":"14","56":"5","57":"7","58":"1","59":"10","60":"8","61":"10","62":"25","63":"NA"},{"1":"1","2":"Student 15","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"80.5809523809524","12":"B-","13":"71.4285714285714","14":"80","15":"78.6666666666667","16":"84","17":"83.3333333333333","18":"NA","19":"13","20":"8","21":"14","22":"4","23":"14","24":"13","25":"9","26":"6","27":"9","28":"34","29":"7","30":"11","31":"6","32":"10","33":"15","34":"4","35":"10","36":"3","37":"7","38":"6","39":"8","40":"8","41":"8","42":"42","43":"9","44":"15","45":"10","46":"15","47":"0","48":"12","49":"12","50":"7","51":"15","52":"13","53":"9","54":"42","55":"12","56":"8","57":"8","58":"0","59":"9","60":"7","61":"9","62":"25","63":"NA"},{"1":"1","2":"Student 16","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"82.1796918767507","12":"B-","13":"72.3809523809524","14":"84.7058823529412","15":"67.3333333333333","16":"91","17":"83.3333333333333","18":"NA","19":"15","20":"10","21":"11","22":"4","23":"10","24":"15","25":"8","26":"7","27":"7","28":"25","29":"10","30":"12","31":"8","32":"10","33":"15","34":"5","35":"10","36":"1","37":"8","38":"7","39":"9","40":"9","41":"8","42":"32","43":"10","44":"15","45":"3","46":"15","47":"5","48":"15","49":"15","50":"10","51":"15","52":"15","53":"10","54":"44","55":"10","56":"4","57":"15","58":"3","59":"9","60":"7","61":"8","62":"25","63":"NA"},{"1":"1","2":"Student 17","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"92.8519140989729","12":"A-","13":"72.3809523809524","14":"98.0392156862745","15":"96","16":"97.7777777777778","17":"93.3333333333333","18":"NA","19":"15","20":"8","21":"13","22":"5","23":"15","24":"15","25":"15","26":"15","27":"7","28":"48","29":"10","30":"13","31":"1","32":"Excused","33":"15","34":"5","35":"10","36":"5","37":"15","38":"4","39":"9","40":"10","41":"9","42":"46","43":"10","44":"15","45":"2","46":"15","47":"5","48":"15","49":"15","50":"10","51":"15","52":"15","53":"10","54":"50","55":"14","56":"10","57":"15","58":"5","59":"10","60":"10","61":"9","62":"28","63":"NA"},{"1":"1","2":"Student 18","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"86.822268907563","12":"B+","13":"74.2857142857143","14":"76.8627450980392","15":"100","16":"89","17":"90","18":"NA","19":"8","20":"8","21":"8","22":"5","23":"6","24":"14","25":"8","26":"15","27":"9","28":"50","29":"7","30":"14","31":"10","32":"10","33":"15","34":"5","35":"8","36":"3","37":"11","38":"3","39":"7","40":"10","41":"10","42":"50","43":"10","44":"15","45":"10","46":"15","47":"0","48":"9","49":"15","50":"5","51":"14","52":"13","53":"9","54":"50","55":"15","56":"7","57":"1","58":"2","59":"9","60":"10","61":"10","62":"27","63":"NA"},{"1":"1","2":"Student 19","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"94.5570028011205","12":"A","13":"69.5238095238095","14":"89.4117647058824","15":"94","16":"97","17":"113.333333333333","18":"NA","19":"11","20":"9","21":"15","22":"4","23":"15","24":"15","25":"14","26":"15","27":"0","28":"50","29":"10","30":"13","31":"0","32":"10","33":"15","34":"5","35":"10","36":"0","37":"2","38":"6","39":"9","40":"10","41":"10","42":"44","43":"8","44":"15","45":"10","46":"15","47":"5","48":"13","49":"15","50":"10","51":"13","52":"14","53":"10","54":"47","55":"13","56":"10","57":"15","58":"5","59":"9","60":"10","61":"10","62":"34","63":"NA"},{"1":"1","2":"Student 20","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"78.8985994397759","12":"C+","13":"64.7619047619048","14":"80.7843137254902","15":"77.3333333333333","16":"88","17":"73.3333333333333","18":"NA","19":"15","20":"8","21":"13","22":"5","23":"9","24":"13","25":"11","26":"7","27":"10","28":"36","29":"10","30":"13","31":"10","32":"9","33":"15","34":"3","35":"10","36":"5","37":"9","38":"4","39":"5","40":"8","41":"8","42":"32","43":"9","44":"13","45":"2","46":"14","47":"0","48":"9","49":"13","50":"7","51":"13","52":"15","53":"9","54":"48","55":"13","56":"6","57":"11","58":"1","59":"8","60":"9","61":"10","62":"22","63":"NA"},{"1":"1","2":"Student 21","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"81.2831932773109","12":"B-","13":"70.4761904761905","14":"76.078431372549","15":"74","16":"92","17":"80","18":"NA","19":"11","20":"10","21":"8","22":"5","23":"10","24":"12","25":"12","26":"9","27":"7","28":"25","29":"7","30":"12","31":"7","32":"10","33":"15","34":"4","35":"10","36":"5","37":"8","38":"8","39":"6","40":"10","41":"8","42":"47","43":"10","44":"15","45":"3","46":"14","47":"0","48":"6","49":"11","50":"9","51":"13","52":"13","53":"10","54":"39","55":"12","56":"4","57":"13","58":"5","59":"9","60":"10","61":"9","62":"24","63":"NA"},{"1":"1","2":"Student 22","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"94.0819327731093","12":"A","13":"91.4285714285714","14":"94.1176470588235","15":"86.6666666666667","16":"95","17":"100","18":"NA","19":"8","20":"9","21":"12","22":"5","23":"15","24":"15","25":"13","26":"15","27":"10","28":"45","29":"10","30":"15","31":"10","32":"10","33":"15","34":"5","35":"10","36":"5","37":"15","38":"9","39":"10","40":"9","41":"8","42":"48","43":"8","44":"13","45":"3","46":"15","47":"5","48":"15","49":"15","50":"10","51":"15","52":"15","53":"10","54":"37","55":"14","56":"10","57":"15","58":"5","59":"10","60":"10","61":"10","62":"30","63":"NA"},{"1":"1","2":"Student 23","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"79.5759103641457","12":"C+","13":"68.5714285714286","14":"78.8235294117647","15":"84.6666666666667","16":"86","17":"73.3333333333333","18":"NA","19":"13","20":"10","21":"14","22":"5","23":"13","24":"14","25":"8","26":"11","27":"10","28":"47","29":"9","30":"15","31":"7","32":"9","33":"15","34":"4","35":"7","36":"5","37":"4","38":"3","39":"6","40":"9","41":"9","42":"40","43":"9","44":"14","45":"2","46":"13","47":"2","48":"13","49":"12","50":"8","51":"15","52":"12","53":"9","54":"40","55":"10","56":"6","57":"5","58":"3","59":"8","60":"10","61":"8","62":"22","63":"NA"},{"1":"1","2":"Student 24","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"89.3633053221289","12":"B+","13":"64.7619047619048","14":"92.5490196078431","15":"99.3333333333333","16":"92","17":"93.3333333333333","18":"NA","19":"15","20":"10","21":"15","22":"5","23":"15","24":"13","25":"15","26":"15","27":"9","28":"49","29":"10","30":"15","31":"3","32":"10","33":"15","34":"4","35":"10","36":"5","37":"13","38":"1","39":"9","40":"9","41":"9","42":"50","43":"9","44":"13","45":"2","46":"15","47":"0","48":"9","49":"15","50":"10","51":"15","52":"14","53":"9","54":"50","55":"15","56":"7","57":"9","58":"4","59":"9","60":"7","61":"9","62":"28","63":"NA"},{"1":"1","2":"Student 25","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"No match","9":"No match","10":"NA","11":"79.2773109243698","12":"C+","13":"70.4761904761905","14":"91.3725490196078","15":"85.3333333333334","16":"92","17":"50","18":"NA","19":"11","20":"10","21":"14","22":"5","23":"15","24":"15","25":"14","26":"14","27":"7","28":"39","29":"9","30":"15","31":"2","32":"10","33":"15","34":"5","35":"10","36":"4","37":"15","38":"9","39":"7","40":"10","41":"10","42":"50","43":"8","44":"15","45":"0","46":"15","47":"1","48":"5","49":"15","50":"5","51":"15","52":"10","53":"9","54":"39","55":"15","56":"10","57":"15","58":"5","59":"10","60":"10","61":"10","62":"15","63":"NA"},{"1":"NA","2":"NA","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"NA","12":"NA","13":"NA","14":"NA","15":"NA","16":"NA","17":"NA","18":"NA","19":"NA","20":"NA","21":"NA","22":"NA","23":"NA","24":"NA","25":"NA","26":"NA","27":"NA","28":"NA","29":"NA","30":"NA","31":"NA","32":"NA","33":"NA","34":"NA","35":"NA","36":"NA","37":"NA","38":"NA","39":"NA","40":"NA","41":"NA","42":"NA","43":"NA","44":"NA","45":"NA","46":"NA","47":"NA","48":"NA","49":"NA","50":"NA","51":"NA","52":"NA","53":"NA","54":"NA","55":"NA","56":"NA","57":"NA","58":"NA","59":"NA","60":"NA","61":"NA","62":"NA","63":"NA"},{"1":"NA","2":"NA","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"NA","12":"NA","13":"NA","14":"NA","15":"NA","16":"NA","17":"NA","18":"NA","19":"NA","20":"NA","21":"NA","22":"NA","23":"NA","24":"NA","25":"NA","26":"NA","27":"NA","28":"NA","29":"NA","30":"NA","31":"NA","32":"NA","33":"NA","34":"NA","35":"NA","36":"NA","37":"NA","38":"NA","39":"NA","40":"NA","41":"NA","42":"NA","43":"NA","44":"NA","45":"NA","46":"NA","47":"NA","48":"NA","49":"NA","50":"NA","51":"NA","52":"NA","53":"NA","54":"NA","55":"NA","56":"NA","57":"NA","58":"NA","59":"NA","60":"NA","61":"NA","62":"NA","63":"NA"},{"1":"NA","2":"NA","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"NA","12":"NA","13":"NA","14":"NA","15":"NA","16":"NA","17":"NA","18":"NA","19":"NA","20":"NA","21":"NA","22":"NA","23":"NA","24":"NA","25":"NA","26":"NA","27":"NA","28":"NA","29":"NA","30":"NA","31":"NA","32":"NA","33":"NA","34":"NA","35":"NA","36":"NA","37":"NA","38":"NA","39":"NA","40":"NA","41":"NA","42":"NA","43":"NA","44":"NA","45":"NA","46":"NA","47":"NA","48":"NA","49":"NA","50":"NA","51":"NA","52":"NA","53":"NA","54":"NA","55":"NA","56":"NA","57":"NA","58":"NA","59":"NA","60":"NA","61":"NA","62":"NA","63":"NA"},{"1":"NA","2":"NA","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"NA","12":"NA","13":"NA","14":"NA","15":"NA","16":"NA","17":"NA","18":"NA","19":"NA","20":"NA","21":"NA","22":"NA","23":"NA","24":"NA","25":"NA","26":"NA","27":"NA","28":"NA","29":"NA","30":"NA","31":"NA","32":"NA","33":"NA","34":"NA","35":"NA","36":"NA","37":"NA","38":"NA","39":"NA","40":"NA","41":"NA","42":"NA","43":"NA","44":"NA","45":"NA","46":"NA","47":"NA","48":"NA","49":"NA","50":"NA","51":"NA","52":"NA","53":"NA","54":"NA","55":"NA","56":"NA","57":"NA","58":"NA","59":"NA","60":"NA","61":"NA","62":"NA","63":"NA"},{"1":"NA","2":"NA","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"NA","12":"NA","13":"NA","14":"NA","15":"NA","16":"NA","17":"NA","18":"NA","19":"NA","20":"NA","21":"NA","22":"NA","23":"NA","24":"NA","25":"NA","26":"NA","27":"NA","28":"NA","29":"NA","30":"NA","31":"NA","32":"NA","33":"NA","34":"NA","35":"NA","36":"NA","37":"NA","38":"NA","39":"NA","40":"NA","41":"NA","42":"NA","43":"NA","44":"NA","45":"NA","46":"NA","47":"NA","48":"NA","49":"NA","50":"NA","51":"NA","52":"NA","53":"NA","54":"NA","55":"NA","56":"NA","57":"NA","58":"NA","59":"NA","60":"NA","61":"NA","62":"NA","63":"NA"},{"1":"NA","2":"NA","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"NA","12":"NA","13":"NA","14":"NA","15":"NA","16":"NA","17":"NA","18":"NA","19":"NA","20":"NA","21":"NA","22":"NA","23":"NA","24":"NA","25":"NA","26":"NA","27":"NA","28":"NA","29":"NA","30":"NA","31":"NA","32":"NA","33":"NA","34":"NA","35":"NA","36":"NA","37":"NA","38":"NA","39":"NA","40":"NA","41":"NA","42":"NA","43":"NA","44":"NA","45":"NA","46":"NA","47":"NA","48":"NA","49":"NA","50":"NA","51":"NA","52":"NA","53":"NA","54":"NA","55":"NA","56":"NA","57":"NA","58":"NA","59":"NA","60":"NA","61":"NA","62":"NA","63":"NA"},{"1":"NA","2":"NA","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"NA","12":"NA","13":"NA","14":"NA","15":"NA","16":"NA","17":"NA","18":"NA","19":"NA","20":"NA","21":"NA","22":"NA","23":"NA","24":"NA","25":"NA","26":"NA","27":"NA","28":"NA","29":"NA","30":"NA","31":"NA","32":"NA","33":"NA","34":"NA","35":"NA","36":"NA","37":"NA","38":"NA","39":"NA","40":"NA","41":"NA","42":"NA","43":"NA","44":"NA","45":"NA","46":"NA","47":"NA","48":"NA","49":"NA","50":"NA","51":"NA","52":"NA","53":"NA","54":"NA","55":"NA","56":"NA","57":"NA","58":"NA","59":"NA","60":"NA","61":"NA","62":"NA","63":"NA"},{"1":"NA","2":"NA","3":"NA","4":"NA","5":"NA","6":"NA","7":"NA","8":"NA","9":"NA","10":"NA","11":"NA","12":"NA","13":"NA","14":"NA","15":"NA","16":"NA","17":"NA","18":"NA","19":"NA","20":"NA","21":"NA","22":"NA","23":"NA","24":"NA","25":"NA","26":"NA","27":"NA","28":"NA","29":"NA","30":"NA","31":"NA","32":"NA","33":"NA","34":"NA","35":"NA","36":"NA","37":"NA","38":"NA","39":"NA","40":"NA","41":"NA","42":"NA","43":"NA","44":"NA","45":"NA","46":"NA","47":"NA","48":"NA","49":"NA","50":"NA","51":"NA","52":"NA","53":"NA","54":"NA","55":"NA","56":"NA","57":"NA","58":"NA","59":"NA","60":"NA","61":"NA","62":"NA","63":"NA"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

#### RStudio file viewer

Finally you could also use RStudio intern file viewer at the console with `View(gradebook)` or programmatically with `view(gradebook)`. The following screenshot display the view via RStudio.

To display the screenshot I used the chunk header together with the `include_graphics()` function of the **{knitr}** package.

````md
```{r rstudio-screenshot-dataset, echo=FALSE, fig.cap='Screenshot of the dataset loaded with `View(ExcelGradeBook)` into RStudio', out.width='95%', fig.align='center', fig.alt='Screenshot of the dataset loaded with `View(ExcelGradeBook)` into RStudio showing that the dataset starts with line 10.', out.extra='class="shadow"'}
knitr::include_graphics("img/inspect-file-with-code-min.png")
```
````


<div class="figure" style="text-align: center">
<img src="img/inspect-file-with-code-min.png" alt="Screenshot of the dataset loaded with `View(ExcelGradeBook)` into RStudio showing that the dataset starts with line 10." width="95%" class="shadow" />
<p class="caption">(\#fig:rstudio-screenshot-dataset)Screenshot of the dataset loaded with `View(ExcelGradeBook)` into RStudio</p>
</div>

#### RStudio import dataset

Using the RStudio interface interactively is another very practical way of inspecting and importing data. In this case you need not to know the exact syntax of the R code. You also can see that there are several datasheets and can inspect their content. When RStudio is importing the dataset it will also write the code lines it has used into the console. You can copy it and use it in your script.


<div class="figure" style="text-align: center">
<img src="img/screenshot-gradebook-rstudio-min.png" alt="Screenshot of the interactive interface of RStudio for importing datasets." width="95%" class="shadow" />
<p class="caption">(\#fig:rstudio-import-data-interface)Screenshot of the interactive interface of RStudio for importing datasets.</p>
</div>

#### Application software

But all these possibilities gives you not the full information. Why are there empty (NA) rows at the end of the dataset? This information you could only get by opening the file with the appropriate software tool. 

<div class="figure" style="text-align: center">
<img src="img/screenshot-gradebook-libre-office-min.png" alt="Screenshot of the dataset loaded into LibreOffice, showing that the dataset starts with line 10 and that at the end is other for the data analysis not relevant information." width="95%" class="shadow" />
<p class="caption">(\#fig:libreoffice-screenshot-dataset)Screenshot of the dataset loaded into LibreOffice</p>
</div>

The screenshot shows that there other datasheets as well, but just the first one is relevant for us. We also get an explication why there are so many NA's rows after the dataset: We mentioned that we used a publicly available gradebook template. It happens that these templates have additional information on their first datasheet how to use the template.

#### Conclusion

The gist of our extensive inspection: 

- Skip the first 10 lines
- Read 26 line into R (25 student and the column headings)
- Use just the first sheet


```r
# Use readxl package to read and import file and assign it a name
ExcelGradeBook <-
  read_xlsx(
    here("data", 
         "gradebooks", 
         "ExcelGradeBook.xlsx"),
    sheet = 1,
    skip = 10,
    n_max = 26
  )

# Copy R object to have a working file and a backup
gradebook <- ExcelGradeBook 
gradebook
#> # A tibble: 25 × 63
#>    Class Name   Race  Gender Age   `Repeated Grade… `Financial Stat… Absent
#>    <dbl> <chr>  <lgl> <lgl>  <lgl> <lgl>            <lgl>            <chr> 
#>  1     1 Stude… NA    NA     NA    NA               NA               1     
#>  2     1 Stude… NA    NA     NA    NA               NA               0     
#>  3     1 Stude… NA    NA     NA    NA               NA               2     
#>  4     1 Stude… NA    NA     NA    NA               NA               0     
#>  5     1 Stude… NA    NA     NA    NA               NA               0     
#>  6     1 Stude… NA    NA     NA    NA               NA               0     
#>  7     1 Stude… NA    NA     NA    NA               NA               0     
#>  8     1 Stude… NA    NA     NA    NA               NA               0     
#>  9     1 Stude… NA    NA     NA    NA               NA               0     
#> 10     1 Stude… NA    NA     NA    NA               NA               <NA>  
#> # … with 15 more rows, and 55 more variables: Late <chr>,
#> #   Make your own categories <lgl>, Running Average <dbl>,
#> #   Letter Grade <chr>, Homeworks <dbl>, Classworks <dbl>,
#> #   Formative Assessments <dbl>, Projects <dbl>,
#> #   Summative Assessments <dbl>, Another Type 2 <lgl>, Classwork 1 <dbl>,
#> #   Homework 1 <dbl>, Classwork 2 <dbl>, Homework 2 <chr>,
#> #   Classwork 3 <chr>, Classwork 4 <dbl>, Classwork 5 <dbl>, …
```


::: {.infobox}
If you have appropriate software (and you know how to use it), then inspect the file as the very first step. It is the easiest way to get the full information about the file to import.
:::
