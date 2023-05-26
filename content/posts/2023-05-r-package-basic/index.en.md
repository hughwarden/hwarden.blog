---
title: "Creating a Package in R - Getting Started"
date: 2023-05-25T21:00:00+01:00
description: "Creating a basic R package from scratch."
slug: "r-package-basic"
tags: ["Package Development", "R"]
series: ["R Package Development"]
series_order: 1
showAuthor: true
showAuthorsBadges : true 
draft: false
---

In my work I code in R a lot and there are a lot of things that I do again and again. One of the very first things you learn when you start programming is that when you have code that you use and write lots of times then you should put it inside a function. The next question though is where do you store this function? If you only need to use it in one project then you can just save it in that script and be done with it. If you want to use these functions in multiple projects then you can save them all in a separate script and source that script at the beginning of your analysis. This works fine, but there is a much better way which is to put all of the functions into a package. What I would like to do in this series of posts is to show how easy it is to create a package and use it in your day to day work. I've made a few packages that are available on CRAN and I am going to write about how to do this, but I'm also going to show how to just host one on GitHub. This is a much less daunting task and is a great way to create reproducible code. This also means that you can do things lke create a package just for you, for example I have an R package that contains functions to change the appearance of ggplot's to my taste. There is no reason for this to be on CRAN, but it's super useful for me to have it on GitHub so I can access it whenever I like and save myself a lot of work.

Now comes my normal disclaimer, I am not an expert in package development but it is something I have done a lot of. What I would like to do is show my workflow that will allow you to create basic R packages. If you would like to know more then check out [this book](https://r-pkgs.org/) by Hadley Wickam that taught me everything I know and has all the answers every time I get stuck. I will be linking to this book a lot in this series.

Last thing before we get started. I am going to be using RStudio to create this package, you don't need to use RStudio but it is designed for this sort of thing and makes it a lot easier. Specifically, I am going to be using a lot of keyboard shortcuts that are specific to RStudio. When I introduce these keyboard shortcuts I will also give you the correpsponding command in R that will do the same thing, if one exists. For some things like documentation there is no command to reproduce this. All this means is that you will have to type out the documentation yourself, which is a little annoying but won't take too long and if you are using an IDE like VSCode then there are plugins that will help you with this (I will not cover them here though). I am also going to be using a Mac to make this package, all of my instructions should work on any system but there might be some slight differences in keyboard shortcuts and file paths, I will try to point these out when I come across them.

## Creating a Package

There are multiple ways to create a package in R, you can do it through the New Project menu or writing the files yourself but my favourite way is to use the `usethis` package. This contains a huge amount of functions to help with making packages in R and I will be using it a lot, so firstly we need to install it. We are also going to be using the `devtools` package a lot so we can install that now as well.

``` r
install.packages("usethis")
install.packages("devtools")
```

`usethis` includes a function called `create_package()` that (surprisingly enough) will create your package for you. In this series I am going to create a package containing some very basic functions, these are so basic that really we don't need to put them in a package but I want to keep the actual content very simple so that we can focus on creating the package itself. I am going to call this package `testpackage`, I recommend that you start off following along the steps in this article exactly and then once we have made it you can play about with it. To get started we can call `create_package()`, this will create a new R project so you do not need to start one first. As an argument we need to give the directory to where we would like to save the package as a string, so in our case this will look like `"<PATH_TO_FOLDER>/testpackage"`. I am going to be saving this in my `Documents` folder so my command will look like this (*this will be different for you*):

``` r
library(usethis)
create_package("~/Documents/testpackage")
```

{{< alert "apple" >}}
Top tip for finding these folder directories on a mac, if you right click on a file and then hold down the option key, the `Copy` option will become `Copy <file> as Pathname` and will copy the path to the file to your clipboard.
{{< /alert >}}


If you are using RStudio, this will open up a new project window at this location. If you look in the file window you will see that a number of files and folders have been created automatically. The good news is that you can ignore most of these, they will be changed and updated automatically and it's actually important that you don't change them by hand. Please do look around but be careful **not** to edit them. As we go along we will create more and more files here but for now we are only going to look at the `DESCRIPTION` file and the `R` folder.

## Adding Metadata to Your Package

Let's open the `DESCRIPTION` file, it should look something like this:

``` r
Package: testpackage
Title: What the Package Does (One Line, Title Case)
Version: 0.0.0.9000
Authors@R: 
    person("First", "Last", , "first.last@example.com", role = c("aut", "cre"),
           comment = c(ORCID = "YOUR-ORCID-ID"))
Description: What the package does (one paragraph).
License: `use_mit_license()`, `use_gpl3_license()` or friends to pick a
    license
Encoding: UTF-8
Roxygen: list(markdown = TRUE)
RoxygenNote: 7.2.3
```

This contains all the information about your package, or metadata. If you upload your package to CRAN then this is the information that will be displayed, it will also be used in various other parts of the process later on. Let's go through each of these sections and see what they mean.

`Package` is just the name of the package, we decided this when we used the `create_package()` function. It's still possible to change the package name, to do so you will need to change it here and also change the name of the folder that contains the package. I don't recommend doing this unless you have a very good reason to do so, also there are rules for naming packages so make sure you follow them (to be safe I would stick with just using letters and you will be fine). Underneath this is `Title`, as the prewritten text says this should be a one line description of what your package does. I'm going to change mine to say `Learning to Create my Own Package`.

Further down, there is a field called `Description` where you can put a short paragraph explaining what your package does (emphasis on short).

### Version Numbers

Then we come along to `Version`. Packages are constantly being updated and changing and we need to keep a track of this. This means that every time we make a change to our package we reflect this by updating the version number. This means that I can keep track of which version of each package I am using for an analysis to make sure that other people will be able to recreate my work exactly. The version number is made up of four numbers and we start at 0.0.0.9000, lets work through these backwards to see what they mean. The last number is called the development number and is probably the one that people are least familiar with. The development number is used to track tiny changes to the package that are not released formally yet. This means that when you are tinkering with the package you will regularly update this number, this version of the package is called the developement version and is generally kept separately to the main package (i.e. if the package is on CRAN then the development version might be kept on GitHub, or if you are hosting the package on GitHub then the deveopment version might be kept on a different branch). People may want to use the development version of the package to have access to the very latest things the package can do, but they do so knowing that things might break or not work properly. The development number, unlike all the others, starts at 9000 but like the others is increased by 1 each time.

The next number is called the patch number, this keeps track of small changes to the package. These are generally small updates that might do something like fix a bug, update documentation or improve error messages so things that don't add anything new and don't break anything old. When you decide you have reached enough changes that you want to release a new (minor) version then this number is increased by one and the development number disappears (for example, your version would go from 0.0.0.9027 to 0.0.1, then when you made your next change to the development version of your package this number would change to 0.0.1.9000).

The next number (the second one from the left) is called the minor number. Generally, we increase this number when we add new functionality to the package. This means that if you are using version 0.1.4 of a package and you upgrade it to 0.2.0 then your code will still work as all the changes are patches or minor changes. When you increase the minor version of a package then the patch number goes back to 0 and the developer number disappears (so if you're on version 0.1.4.9042 and you increase the minor version then you will go to 0.2.0, when you then make a change to the development version of you package then the next change to your version number will be to 0.2.0.9000).

The final number (or the first one from the left) is the major version. We change the major version of our package when we make changes that might break code people have written using our package. For example, if you write code using 0.2.4 of a package and then you update the package to 1.0.0 then your code might no longer work. This might mean that you decide not to make this update for that project or you go back and check your code separately. It's important to note here, that if a major version of a package changes and your code still runs **this does not necessarily mean that your code is still working properly**, you still have to check that your result is being calculated how you expect it to as the package may be using a different method or have different presets. This is all a long way of saying, leave the version number alone for now.

### Authors

Then we get on to the `Person` field. Here you should fill in all of your information, this will be used to identify you as the creator of the package, cite you if your package is used for any research and contact you if there is something wrong with the package. If you aren't submitting this package to CRAN, I recommend not putting an email address here and people can contact you through GitHub Issues, if you are, then I recommend using either an email specifically for dealing with package development related issues or another public email address as anyone will be able to access this. You can see under `role` there are two three-letter codes, these identify you as the *aut*hor and *cre*ator of the package. There are lots of these, you can find commonly used ones [here](https://r-pkgs.org/description.html#sec-description-authors-at-r).

There is also then a space for you to input your ORCID ID. If you do not have one then I recommend you create one but you can also just remove the `comment` argument.

### Licenses

Now we have come to the legal part and the obligatory *I am not a lawyer, there may be slight mistakes in what I have written below*. I very much suggest you read up fully about licenses, you can do that [here](https://r-pkgs.org/license.html). There are a few different types of licenses, I have only ever used 3 and they have always met all my needs. The license tells people what they are allowed to do with your code (not with your package, but the actual code you have written). It also protects you by clearly providing your software "without warranty", essentially meaning if your code doesn't work properly and they lose money then they can't come back and sue you.

The first type of license is the MIT license, this allows anybody to use your code for anything they want to, as long as they credit you. The second type is the GPL3 license that is the same as MIT except that if they use your code to make some software then that software also has to be published under the GPL3 license and must be made open source (meaning it must be released for free!). The final license is a CC0 license, I don't have as much experience with this one, but it seems to be like the MIT license except that they don't have to credit you (i.e. it just becomes code owned by everyone).

In the past, I have used MIT licenses for small packages and packages I have made for myself, GPL3 licenses for the more serious/bigger packages that I have made and CC0 licenses for teaching material I have contributed to. Each of these licenses can be added to the package using a function from `usethis`, after you have chosen the license you want then you can **save your changes** to `DESCRIPTION` and then run one of the following

```r
library(usethis)

use_mit_license()
use_gpl3_license()
use_cc0_license()
```

I am going to use the MIT license for this package, so I will run `use_mit_license()` and then save my changes to `DESCRIPTION`. You can now see that the license has been added to the `DESCRIPTION` file and you have two new files in your package, `LICENSE` and `LICENSE.md`, do read these but you do not need to change them. If you look at the license, you can see the copyright is attributed to `testpackage authors`, if you would like specify exactly who you are you can run `use_mit_license("Your Name")`. If you have already created your license but would like to change it to use your name then you can run the function again with your name as an argument and select yes to overwrite what is already there. If you release your package and then decide you would like to use a different license then this can be really difficult to change, so make sure you think about it before you choose. In most cases it doesn't really matter and so I'd just pick MIT or GPL3.

Don't touch the last three lines, save `DESCRIPTION` and then you are done! Your file should look something like this:

```r
Package: testpackage
Title: Learning to Create my Own Package
Version: 0.0.0.9000
Authors@R: 
    person("Hugh", "Warden", , "my_email@example.com", role = c("aut", "cre"),
           comment = c(ORCID = "1234-12345-12"))
Description: A collection of functions that I find useful that I have collected 
    together to learn how to make a package in R.
License: MIT + file LICENSE
Encoding: UTF-8
Roxygen: list(markdown = TRUE)
RoxygenNote: 7.2.3
```

The `DESCRIPTION` file will change as we go along so we will come back and look at it at the end, but for now we are done and we won't have to manually change it again.

## Adding Functions

Now we are going to start adding functions to our package. To add functions we go through something called the Load > Document > Test cycle. I am only going to go through the Load part of this cycle here and the Document and Test parts will be covered in their own sections. The Load part of the cycle is where we actually add the function to the package, so at the end of this you will have a working package. Functions are stored in R scripts inside the `R` folder. We can create an R script using a `usethis` function:

``` r
use_r("greeting")
```

### Add a Simple Function

If you run this function you will see there is now an R script inside the `R` directory called `greeting.R`. The name of this file doesn't matter, we can put as many R scripts as we like in this folder and call them what we like. However, if we have lots of functions it would be messy to have them all in one R script and so we can group similar functions in different scripts to keep things tidy. I have called this file greeting because we are going to put some functions in there that will print a greeting. Inside `greeting.R` write the following function:

``` r
my_name <- function() {
  "Hugh"
}
```

Replace `"Hugh"` with your name and you now have a function that will return your name, how exciting! Normally to use this function we would have to run this script, but we don't do this when making a package. Instead we use a function called `load_all()` from the `devtools` package (hence why this is called the Load step). We can do it like this

``` r
library(devtools)

load_all()
```

When you make a package you will use this function a lot, so in RStudio it has a handy shortcut which is cmd/shift/L (or ctrl/shift/L not on a mac). Once you have written your function and run `load_all()` you will then be able to type `my_name()` into the console and your name should print out.

### Add a Function That Uses Another Function

So now we have a very simpe function that return our name. Let's start looking at slightly more complicated functions. Now we are going to write a function that returns a greeting, we will add this below the `my_name()` function in `greeting.R` (we could have put it in another script if we wanted, but I didn't want to):

``` r
greeting <- function() {
  paste0("Hello, my name is ", my_name())
}
```

Here we have written a function that will print a greeting. If you run `load_all()` you will then be able to use this function straight away. This function is a bit more complicated as it uses two other functions, `paste0()` and `my_name()`. `paste0()` is a base R function that will paste together strings, the `0` means that it will not add any spaces between the strings. The `my_name()` function is the one we wrote above. If you run `greeting()` then you should see your greeting printed out. Normally you have to do something special to use other functions inside your package, but there are two cases where you don't (1) when the function you are using is a base R function (this is a function that you don't have to load the library for, like `paste0`) or (2) when the function you are using is another function in the package (like `my_name`).

### Add a Function That Uses a Package

Now what if you want to use a function from another package? You can't run `library` to load other packages inside your own package, instead you have to use `::`. This is called the double colon operator and it allows you to access functions from other packages. For example, if you wanted to use the `ggplot` function from `ggplot2` you would write `ggplot2::ggplot()`. We are going to use the `stringr` package to write a function that will return the number of characters in your name. To do this though, we need to make a note somewhere in the package that we want to use the `stringr` package. This is done in the `DESCRIPTION` file, but there is a function that will do this for us:

``` r
use_package("stringr")
```

If you run this function then you will see that the `DESCRIPTION` file has been updated to include `stringr` in the `Imports` section. This means that we can now use any function from `stringr` in our package using the '::' operator. Now we can add the following function to `greeting.R`:

``` r
name_length <- function() {
  stringr::str_length(my_name())
}
```

If you now run `load_all()` you can run this function and it will return the number of characters in your name. Using the double colon operator can be a bit annoying but it is really important to make sure that your code is unambiguous.

### Add Arguments to a Function

At the moment none of our functions take any arguments. This is fine but later on we will want to look at functions that have arguments, therefore I am going to change the `greeting()` and `name_length()` functions to take an argument. The argument will be called `name`, it will be set to `NULL` by default and if it is `NULL` then it will use the `my_name()` function. If it is not `NULL` then it will use the name that is passed to it. This is what the functions will look like:

``` r
greeting <- function(name = NULL) {
  if (is.null(name)) {
    name <- my_name()
  }
  paste0("Hello, my name is ", name)
}

name_length <- function(name = NULL) {
  if (is.null(name)) {
    name <- my_name()
  }
  stringr::str_length(name)
}
```

## Wrap Up

We have now created the bare bones of our package. We have created the package structure and added some functions to it. Over the next series of posts we will first look at how to document our functions and then we will look at how to test those functions. After that there will be another post on how to install the package locally, how to install it through GitHub and then a little bit on how to submit your package to CRAN, this might also include small bits on continuous integration and extra documentation like README's, NEWS, vignettes and how these can all automatically generate a website for your package.