R Client for Dataverse Repositories
================

[![CRAN
Version](https://www.r-pkg.org/badges/version/dataverse)](https://cran.r-project.org/package=dataverse)
![Downloads](https://cranlogs.r-pkg.org/badges/dataverse) [![Travis-CI
Build
Status](https://travis-ci.org/IQSS/dataverse-client-r.png?branch=master)](https://travis-ci.org/IQSS/dataverse-client-r)
[![codecov.io](https://codecov.io/github/IQSS/dataverse-client-r/coverage.svg?branch=master)](https://codecov.io/github/IQSS/dataverse-client-r?branch=master)

[![Dataverse Project
logo](https://dataverse.org/files/dataverseorg/files/dataverse_project_logo-hp.png)](https://dataverse.org)

The **dataverse** package provides access to
[Dataverse](https://dataverse.org/) APIs (versions 4+), enabling data
search, retrieval, and deposit, thus allowing R users to integrate
public data sharing into the reproducible research workflow.
**dataverse** is the next-generation iteration of [the **dvn**
package](https://cran.r-project.org/package=dvn), which works with
Dataverse 3 (“Dataverse Network”) applications. **dataverse** includes
numerous improvements for data search, download, and deposit.

### Getting Started

You can find a stable release on
[CRAN](https://cran.r-project.org/package=dataverse), or install the
latest development version from
[GitHub](https://github.com/iqss/dataverse-client-r/):

``` r
# Install from CRAN
install.packages("dataverse")

# Install from GitHub
# install.packages("remotes")
remotes::install_github("iqss/dataverse-client-r")
```

#### Keys

Some features of the Dataverse API are public and require no
authentication. This means in many cases you can search for and retrieve
data without a Dataverse account for that a specific Dataverse
installation. But, other features require a Dataverse account for the
specific server installation of the Dataverse software, and an API key
linked to that account. Instructions for obtaining an account and
setting up an API key are available in the [Dataverse User
Guide](https://guides.dataverse.org/en/latest/user/account.html). (Note:
if your key is compromised, it can be regenerated to preserve security.)
Once you have an API key, this should be stored as an environment
variable called `DATAVERSE_KEY`. It can be set within R using:

``` r
Sys.setenv("DATAVERSE_KEY" = "examplekey12345")
```

where `examplekey12345` should be replace with your own key.

#### Server

Because [there are many Dataverse
installations](https://dataverse.org/), all functions in the R client
require specifying what server installation you are interacting with.
This can be set by default with an environment variable,
`DATAVERSE_SERVER`. This should be the Dataverse server, without the
“https” prefix or the “/api” URL path, etc. For example, the Harvard
Dataverse can be used by setting:

``` r
Sys.setenv("DATAVERSE_SERVER" = "dataverse.harvard.edu")
```

Note: The package attempts to compensate for any malformed values,
though.

Currently, the package wraps the data management features of the
Dataverse API. Functions for other API features - related to user
management and permissions - are not currently exported in the package
(but are drafted in the [source
code](https://github.com/IQSS/dataverse-client-r)).

### Data Download

The dataverse package provides multiple interfaces to obtain data into
R. Users can supply a file DOI, a dataset DOI combined with a filename,
or a dataverse object. They can read in the file as a raw binary or a
dataset read in with the appropriate R function.

#### Reading data as R objects

Use the `get_dataframe_*()` functions, depending on the input you have.
For example, we will read a survey dataset on Dataverse,
[nlsw88.dta](https://demo.dataverse.org/dataset.xhtml?persistentId=doi:10.70122/FK2/PPIAXE)
(`doi:10.70122/FK2/PPKHI1/ZYATZZ`), originally in Stata dta form.

With a file DOI, we can use the `get_dataframe_by_doi` function:

``` r
nlsw <-
  get_dataframe_by_doi(
    filedoi     = "10.70122/FK2/PPIAXE/MHDB0O",
    server      = "demo.dataverse.org"
  )
```

    ## Downloading ingested version of data with readr::read_tsv. To download the original version and remove this message, set original = TRUE.

    ## 
    ## ── Column specification ────────────────────────────────────────────────────────────────────────────────────────────────
    ## cols(
    ##   idcode = col_double(),
    ##   age = col_double(),
    ##   race = col_double(),
    ##   married = col_double(),
    ##   never_married = col_double(),
    ##   grade = col_double(),
    ##   collgrad = col_double(),
    ##   south = col_double(),
    ##   smsa = col_double(),
    ##   c_city = col_double(),
    ##   industry = col_double(),
    ##   occupation = col_double(),
    ##   union = col_double(),
    ##   wage = col_double(),
    ##   hours = col_double(),
    ##   ttl_exp = col_double(),
    ##   tenure = col_double()
    ## )

which by default reads in the ingested file (not the original dta) by
the
[`readr::read_tsv`](https://readr.tidyverse.org/reference/read_delim.html)
function.

Alternatively, we can download the same file by specifying the filename
and the DOI of the “dataset” (in Dataverse, a collection of files is
called a dataset).

``` r
nlsw_tsv <-
  get_dataframe_by_name(
    filename  = "nlsw88.tab",
    dataset   = "10.70122/FK2/PPIAXE",
    server    = "demo.dataverse.org"
  )
```

Now, Dataverse often translates rectangular data into an ingested, or
“archival” version, which is application-neutral and easily-readable.
`read_dataframe_*()` defaults to taking this ingested version rather
than using the original, through the argument `original = FALSE`.

This default is safe because you may not have the proprietary software
that was originally used. On the other hand, the data may have lost
information in the process of the ingestation.

Instead, to read the same file but its original version, specify
`original = TRUE` and set an `.f` argument. In this case, we know that
`nlsw88.tab` is a Stata `.dta` dataset, so we will use the
`haven::read_dta` function.

``` r
nlsw_original <-
  get_dataframe_by_name(
    filename    = "nlsw88.tab",
    dataset     = "10.70122/FK2/PPIAXE",
    .f          = haven::read_dta,
    original    = TRUE,
    server      = "demo.dataverse.org"
  )
```

Note that even though the file prefix is “.tab”, we use
`haven::read_dta`.

Of course, when the dataset is not ingested (such as a Rds file), users
would always need to specify an `.f` argument for the specific file.

Note the difference between `nls_tsv` and `nls_original`. `nls_original`
preserves the data attributes like value labels, whereas `nls_tsv` has
dropped this or left this in file metadata.

``` r
class(nlsw_tsv$race) # tab ingested version only has numeric data
```

    ## [1] "numeric"

``` r
attr(nlsw_original$race, "labels") # original dta has value labels
```

    ## white black other 
    ##     1     2     3

### Data Archiving

Dataverse provides two - basically unrelated - workflows for managing
(adding, documenting, and publishing) datasets. The first is built on
[SWORD v2.0](http://swordapp.org/sword-v2/). This means that to create a
new dataset listing, you will have to first initialize a dataset entry
with some metadata, add one or more files to the dataset, and then
publish it. This looks something like the following:

``` r
# After setting appropriate dataverse server and environment, obtain SWORD
# service doc
d <- service_document()

# create a list of metadata for a file
metadat <-
  list(
    title       = paste0("My-Study_", format(Sys.time(), '%Y-%m-%d_%H:%M')),
    creator     = "Doe, John",
    description = "An example study"
  )

# create the dataset, where "mydataverse" is to be replaced by the name 
# of the already-created dataverse as shown in the URL
ds <- initiate_sword_dataset("<mydataverse>", body = metadat)

# add files to dataset
readr::write_csv(iris, file = "iris.csv")

# Search the initiated dataset and give a DOI and version of the dataverse as an identifier
mydoi <- "doi:10.70122/FK2/BMZPJZ&version=DRAFT"

# add dataset
add_dataset_file(file = "iris.csv", dataset = mydoi)

# publish new dataset
publish_sword_dataset(ds)

# dataset will now be published
list_datasets("<mydataverse>")
```

The second workflow is called the “native” API and is similar but uses
slightly different functions:

``` r
# create the dataset
ds <- create_dataset("mydataverse")

# add files
tmp <- tempfile()
write.csv(iris, file = tmp)
f <- add_dataset_file(file = tmp, dataset = ds)

# publish dataset
publish_dataset(ds)

# dataset will now be published
get_dataverse("mydataverse")
```

Through the native API it is possible to update a dataset by modifying
its metadata with `update_dataset()` or file contents using
`update_dataset_file()` and then republish a new version using
`publish_dataset()`.

For more extensive features of updating and maintaining data, see
[pyDataverse](https://pydataverse.readthedocs.io/en/latest/).

### Related Software

Other dataverse clients include
[pyDataverse](https://pydataverse.readthedocs.io/en/latest/) for Python
and the [Java client](https://github.com/IQSS/dataverse-client-java).

Users interested in downloading metadata from archives other than
Dataverse may be interested in Kurt Hornik’s
[OAIHarvester](https://cran.r-project.org/package=OAIHarvester) and
Scott Chamberlain’s [oai](https://cran.r-project.org/package=oai), which
offer metadata download from any web repository that is compliant with
the [Open Archives Initiative](http://www.openarchives.org/) standards.
Additionally, [rdryad](https://cran.r-project.org/package=rdryad) uses
OAIHarvester to interface with [Dryad](https://datadryad.org/stash). The
[rfigshare](https://cran.r-project.org/package=rfigshare) package works
in a similar spirit to **dataverse** with <https://figshare.com/>.
