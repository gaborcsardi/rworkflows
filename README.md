r\_workflows
================
<h4>
Authors:
<h4>
<h5>
Brian M. Schilder and Alan Murphy
</h5>
<h5>
<a href='https://www.neurogenomics.co.uk/' target='_blank'>Neurogenomics
Lab</a>
</h5>
<h5>
<a href='https://www.imperial.ac.uk/brain-sciences/' target='_blank'>Imperial
College London</a>
</h5>
<h4>
Most recent update:
</h4>
<h5>
Oct-28-2021
</h5>

## Intro

[GitHub Actions](https://docs.github.com/en/actions) are a powerful way
to automatically launch workflows every time you push changes to a
GitHub repository. This is a form of [Continuous Integration
(CI)](https://docs.github.com/en/actions/automating-builds-and-tests/about-continuous-integration),
which helps ensure that your code is always working as expected (without
having to manually check each time).

Here, we have designed robust and flexible workflows specifically for
the development of R packages.

Currently, these workflows perform the following tasks:

1.  Build a Docker container to run subsequent steps within.
2.  Build and check your R package (with CRAN and Bioconductor
    checks).  
3.  Run units tests.  
4.  Run code coverage tests and upload the results to
    [Codecov](https://about.codecov.io/).  
5.  (Re)build and launch a documentation website for your R package.  
6.  Push an [Rstudio](https://www.rstudio.com/)
    [Docker](https://www.docker.com/) container to
    [DockerHub](https://hub.docker.com/) with your package and all its
    dependencies pre-installed.

Importantly, these files are designed to work with any R package
out-of-the-box. This means you won’t have to manually edit any of the
files provided here, simply copy and paste them into the appropriate
folders.

## Setup

If you haven’t already [created your R
package](https://support.rstudio.com/hc/en-us/articles/200486488-Developing-Packages-with-the-RStudio-IDE),
you may want to get started with
[`devtools`](https://github.com/r-lib/devtools) and/or
[`biocthis`](https://github.com/lcolladotor/biocthis) which include
functions for generating R package templates. However as long as you
have the basic components of an R package (e.g. `DECSRIPTION`.
`R/<functions.R>`, etc.) stored on a GitHub repo, the workflows provided
here will work.

To use these workflows when developing your own R package, follow these
steps (after replacing text indicated by `< >`):

### 1. Setup GitHub Secrets

GitHub now lets you store “Secrets” within each repo. These are
basically hidden variables that no one but the repo admins can see/edit.

See here for instructions on how to [create GitHub
Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets).

You will need to store the following variables as Secrets in your GitHub
repo.

-   `PAT_GITHUB`: &lt;Your GitHub Personal Authentication Token
    generated [via the following
    instructions](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)&gt;  
-   `DOCKER_USERNAME`: &lt;Your DockerHub username. [Create an
    account](https://hub.docker.com/signup) if you haven’t already&gt;  
-   `DOCKER_TOKEN`: &lt;Your DockerHub access token generated [via the
    following
    instructions](https://docs.docker.com/docker-hub/access-tokens/)&gt;  
-   `CODECOV_TOKEN`: &lt;A [Codecov](https://app.codecov.io/gh) token
    generated by visiting the link:
    `https://app.codecov.io/gh/<github_username>/<github_repo>` &gt;

### 2. Copy the relevant folders/files

The easiest way to download the necessary folders/files is by cloning
this repo.

    git clone https://github.com/neurogenomics/r_workflows.git

Copy the following folders/files into the folder where your R package’s
project folder.

    scp -r r_workflows/.github <my_package_folder>
    scp r_workflows/Dockerfile <my_package_folder>

Alternatively, you can simply download the files you need directly from
your web browser and place them within your R project folder.

**NOTE**: When copying files, you will need to use the exact folder
structure used in this repo, including the fact that `.github/` starts
with a `.`. This means it will be a “hidden” folder, but you can still
view it within the “Files” window in Rstudio, or by running `ls -lah`
within the command line.

## Usage

Now, these workflows will automatically be run every time you push
changes to your GitHub repo.

    git add .
    git commit -m "<commit message>"
    git push

You can then monitor the progress and view the results of your workflows
via the *“Actions”* tab of your GitHub repo.

**NOTE**: If you do not see an *“Actions”* tab, go to *Settings –&gt;
Actions* and make sure “Actions permissions” is activated.

## File descriptions

### `.github/workflows/check-bioc-docker.yml`

Steps 1-5 [above](https://github.com/neurogenomics/r_workflows#Intro).

This workflow is derived from the workflow generated by the
[`use_bioc_github_action()`](https://lcolladotor.github.io/biocthis/articles/biocthis.html)
function within the
[`biothis`](http://www.bioconductor.org/packages/release/bioc/html/biocthis.html)
package. This workflow takes advantage of several key features:

-   Uses the official
    [`Bioconductor/bioconductor_docker`](https://github.com/Bioconductor/bioconductor_docker)
    Docker container (to minimise errors due to versioning
    conflicts/missing dependencies).  
-   Uses the package
    [AnVIL](https://bioconductor.org/packages/release/bioc/html/AnVIL.html)
    to more rapidly install R packages from binaries.

However, I have modified the `bioc` workflow to make a number of
improvements:

-   Uses dynamic variables to specify R/Bioconductor versions
    (e.g. `r: "latest"`) and the name of your R package, as opposed to
    static names that are likely to become outdated
    (e.g. `r: "4.0.1"`).  
-   Additional error handling and dependencies checks.  
-   Re-renders `README.Rmd` before rebuilding the documentation website.

### `.github/workflows/dockerhub.yml`

Step 6 [above](https://github.com/neurogenomics/r_workflows#Intro).

Uses the official
[`Bioconductor/bioconductor_docker`](https://github.com/Bioconductor/bioconductor_docker)
Docker container.

**NOTE**: The `Bioconductor/bioconductor_docker` container often lags
behind the actual Bioconductor releases. This means that sometimes
“devel” in `Bioconductor/bioconductor_docker` is actually referring to
the “release” version of Bioconductor. See this
[Issue](https://github.com/Bioconductor/bioconductor_docker/issues/37)
for details.

### `Dockerfile`

### Key features

This Dockerfile is designed for developers of any R package stored on
GitHub. Unlike other Dockerfiles, this one **does not require any manual
editing when applying to different R packages**. This means that users
who are unfamiliar with Docker do not have to troubleshoot making this
file correctly. It also means that it will continue to work even if your
R package dependencies change.

It runs several steps:

1.  Pulls the official bioconductor Docker container (which includes
    Rstudio).  
2.  Runs CRAN checks on the R package.  
3.  Runs all Bioconductor checks on the R package.  
4.  Installs the R package and all of its dependencies (including
    `Depends`, `Imports`, and `Suggests`).

**NOTE**: If any of the CRAN/Bioconductor checks do not pass, the Docker
container will not be pushed to DockerHub. This means you don’t have to
worry about a faulty version of your package accidentally being deployed
to DockerHub.

This Dockerfile should be used with the
[`dockerhub.yml`](https://github.com/neurogenomics/r_workflows/blob/master/.github/workflows/dockerhub.yml)
workflow file, as you must first checkout the R package from GitHub,
along with several other GitHub Actions.

If the R package passes all checks, the `dockerhub.yml` workflow will
subsequently push the Docker container to DockerHub (using the username
and token credentials stored as GitHub Secrets).

## Docker usage

You can then create an image of the Docker container in any command
line:

    docker pull <DockerHub_repo_name>/<package_name>

Once the image has been created, you can launch it with:

    docker run \
        -d \
        -e ROOT=true \
        -e PASSWORD=bioc \
        -v ~/Desktop:/Desktop \
        -v /Volumes:/Volumes \
        --rm \
        -p 8788:8787 <DockerHub_repo_name>/<package_name> 

**NOTES**:  
\* The `-d` ensures the container will run in “detached” mode, which
means it will persist even after you’ve closed your command line
session.  
\* Optionally, you can also install the [Docker
Desktop](https://www.docker.com/products/docker-desktop) to easily
manage your containers.  
\* You can set the password to whatever you like by changing the
`-e PASSWORD=...` flag.  
\* The username will be *“rstudio”* by default.

Finally, launch the containerised Rstudio by entering the following URL
in any web browser:

<http://localhost:8788/>

## Singularity usage

If you are using a system that does not allow Docker (as is the case for
many institutional computing clusters), you can instead [install Docker
images via
Singularity](https://sylabs.io/guides/2.6/user-guide/singularity_and_docker.html).

    singularity pull docker:://<DockerHub_repo_name>/<package_name>
    singularity run <package_name>_latest.sif

# Session Info

<details>

``` r
utils::sessionInfo()
```

    ## R version 4.1.0 (2021-05-18)
    ## Platform: x86_64-apple-darwin17.0 (64-bit)
    ## Running under: macOS Big Sur 10.16
    ## 
    ## Matrix products: default
    ## BLAS:   /Library/Frameworks/R.framework/Versions/4.1/Resources/lib/libRblas.dylib
    ## LAPACK: /Library/Frameworks/R.framework/Versions/4.1/Resources/lib/libRlapack.dylib
    ## 
    ## locale:
    ## [1] en_GB.UTF-8/en_GB.UTF-8/en_GB.UTF-8/C/en_GB.UTF-8/en_GB.UTF-8
    ## 
    ## attached base packages:
    ## [1] stats     graphics  grDevices utils     datasets  methods   base     
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] compiler_4.1.0  magrittr_2.0.1  fastmap_1.1.0   tools_4.1.0    
    ##  [5] htmltools_0.5.2 yaml_2.2.1      stringi_1.7.5   rmarkdown_2.11 
    ##  [9] knitr_1.36      stringr_1.4.0   xfun_0.27       digest_0.6.28  
    ## [13] rlang_0.4.12    evaluate_0.14

</details>
