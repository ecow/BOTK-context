# BOTK\Context
[![Build Status](https://img.shields.io/travis/linkeddatacenter/BOTK-context.svg?style=flat-square)](http://travis-ci.org/linkeddatacenter/BOTK-context)
[![Code Coverage](https://img.shields.io/scrutinizer/coverage/g/linkeddatacenter/BOTK-context.svg?style=flat-square)](https://scrutinizer-ci.com/g/linkeddatacenter/BOTK-context)
[![Latest Version](https://img.shields.io/packagist/v/botk/context.svg?style=flat-square)](https://packagist.org/packages/botk/context)
[![Total Downloads](https://img.shields.io/packagist/dt/botk/context.svg?style=flat-square)](https://packagist.org/packages/botk/context)
[![License](https://img.shields.io/packagist/l/botk/context.svg?style=flat-square)](https://packagist.org/packages/botk/context)

Ultraligth validation and sanitization helpers.

This is a BOTK package. Please refer to http://ontology.it/tools/botk for more info
about BOTK project.

## Quick start

The package is available on [Packagist](https://packagist.org/packages/botk/context).
You can install it using [Composer](http://getcomposer.org).

```bash
composer require botk/context
```

Some code examples in samples directory.

## BOTK\Context package documentation


### Abstract

This package exports a set of class to manage the context of a RESTful request.

A Context is defined as the full set of variables available runtime durin the serving of an http request.
          Variables are grouped in distinct name spaces.

All variables in name spaces are **read only**.

This package exposes the methods to sanitize and validate context variables.

Note that cookies and session aren't part of context. This because proper RESTful resource implementation
          shouldn't need persisten status in any way.


### Installation

This package follows[ BOTK guide line for installation](../overview/#installation) and require [composer](http://getcomposer.org/).

Add following dependances to **composer.json** file in your project root:

    {
        "require": {
            "botk/context": "*"
        }
    }

### The Context class

A Context is the full set of all that a CGI endpoint knows about&nbsp; Request beside its URI. Context space
        contains system environment variables, requests header, request body, server specific vars, cookies, variables
        in configuration files, local define variables, etc etc..

Context variables are grouped in _namespaces_.` BOTK\Context\Context` class provide the
        method&nbsp; ` ns(<var>namespace name</var>)` to access name spaces. There are five predefined
        namespaces:

1.  the variables defined in the heading of the request and exposed by web server (<var>INPUT_SERVER</var>); 
3.  the variables encoded in http URI (<var>INPUT_GET</var><var></var>); 
5.  the variables encoded in http body (<var>INPUT_POST</var>);
7.  the system environment (<var>INPUT_ENV</var>); 
9.  local defined variables (Context::LOCAL).


Beside these, Context class allow you to define dynamic namespace built from .ini files loaded runtime from a
configuration directory. The name of the namespace is the name of the configuration file without .ini extension.

The configuration file must exists when is first referenced. The default configuration directory is
        ../configs, that normally is a directory placed at the same level of the directory that contains the scripts who
        instance a context class. You can change the default directory defining the environment variable $_ENV['BOTK_CONFIGDIR'].

Context shold be considered as a readonly data structure because you cant change the resource execution context.

For example suppose to have a end-point script is in ` myapp/httdocs/index.php ` so, to get a
        success on following code, it must exist ` myapp/configs/sample.ini ` file:

    use BOTK\Context\Context as CX;
    $sampleNameSpace = CX::factory()->ns('sample');

If you want to put the .ini file in the same end-point directory:

    use BOTK\Context\Context as CX;
    $_ENV['BOTK_CONFIGDIR'] = '.';
    $sampleNameSpace = CX::factory()->ns('sample');

You can also access to local variable passing them to context constructor:

    $myvar = 'ok'; // define a variable in local scope
    $v = CX::factory(get_defined_vars())->ns(CX::LOCAL)->getValue('myvar');
    //$v == 'ok'

Context class exposes the method `guessRequestCanonicalUri()` that returns the current request uri
        in canonical form and the method `guessRequestRelativelUri()` that returns it as relative path.

Please note that get request uri is not a trivial job, this because http servers and proxyes can change what
        the user entered. You can just guess about it.

### The ContextNameSpace Class

This class implements the controlled access to set of variables.

#### The getValue() method

The `BOTK\Context\ContextNameSpace` class exports the method ` getValue() ` that
        allows to validate and sanitize variables defined in a namespace (see [PHP
          Validation & Sanitization](http://foaa.de/blog/2012/11/27/php-validation-and-sanitization/) article).

If somethings goes wrong it throwns an error that can be trapped by standard BOTK error management.

`getValue()` exposes following interface:

In addition to `getValue()``ContextNameSpace` class expose a set of shortcuts for
        easy to read and write code.

### validator shortcucts

Some validators shold be boring to write, here are a set of predefined shortcuts:

### getValue shortcucts

Even with validator shortcuts, a call to getValue method can be too verbose, here are some shortcut you can use
        for common task:

*  getString($varName,$default = null):&nbsp; same as
              getValue($varName,$default, null, FILTER_SANITIZE_STRING);
*  getURI($varName,$default = null):&nbsp; same as
              getValue($varName,$default, self::STRING('/.+/'), FILTER_SANITIZE_URL)

### PagedResourceContext class

The PagedResource class is a facility to manage the context of a resource whose query string can contains
        variables to drive pagination processing. It is a specialization of Context class and like Context should be
        considered as a read-only data structure.

The resource paging context is inspired on W3C's Linked Data Platform Paging specifications .Here are some imported definitions:
<dl class="glossary"><dt><dfn>Paged resource</dfn></dt><dd>A resource&nbsp; whose representation may be too large to fit in a single HTTP response, for which a server
          offers a sequence of single-page resources. A paged <var>P</var> is broken into a sequence of pages
          (single-page resources) <var>P<sub>1</sub>, P<sub>2</sub>, ...,P<sub>n</sub></var>, the representation of
          each <var>P<sub>i</sub></var> contains a subset of&nbsp; <var>P</var>.. </dd><dt><dfn>Single-page resource</dfn></dt><dd>One of a sequence of related resources <var>P<sub>1</sub>, P<sub>2</sub>, ...,P<sub>n</sub></var>, each of
          which contains a subset of the state of another resource <var>P</var>. <var>P</var> is called the paged
          resource. For readers familiar with paged feeds, a single-page resource is similar to a feed document and the
          same coherency/completeness considerations apply.

Note: the choice of terms was designed to help authors and readers clearly differentiate between the <a title="Paged resource">_resource
                being paged_</a>, and the <a title="Single-page resource">_individual page resources_</a>,
            in cases where both are mentioned in close proximity.

</dd><dt><dfn>first page 
</dfn></dt><dd>The uri to the first <a style="text-decoration: underline;" title="Single-page resource">single-page
            resource</a> of a <a title="Paged resource">paged resource</a><var>P</var>. For example
          http://www.example.org/bigresource?page=0 </dd><dt><dfn>next page 
</dfn></dt><dd>The uri&nbsp; to the next <a title="Single-page resource">single-page resource</a> of a <a title="Paged resource">paged
            resource</a><var>P</var>. </dd><dt><dfn>last page
</dfn></dt><dd>The uri&nbsp; to the last <a title="Single-page resource">single-page resource</a> of a <a title="Paged resource">paged
            resource</a><var>P</var>. </dd><dt><dfn>previous page 
</dfn></dt><dd>The uri&nbsp; to the previous <a title="Single-page resource">single-page resource</a> of a <a title="Paged resource">paged
            resource</a><var>P</var>. </dd></dl>

Pagination require some variable to be specified in resource URI query strings. Such variables name and other
        default values can be passed to class constructor as an option associative array. Here are supported keywords:

*   plabel : contains the variable that contains page num. For default 'page'
*   pslabel&nbsp; : contains the variable that contains page size. For default 'pagesize
*   pagesize&nbsp; : contains the default value for page size. The default is 100

It exposes following methods:

<dl><dt>`getSinglePageResourceUri( $pagenum = null, $pagesize=null)`</dt><dd>returns the canonical uri with page variables in query string. Without arguments returns current page
          resource uri</dd><dt>`getPagedResourceUri()`</dt><dd>return the resource uri wit page variables removed from query string&nbsp; </dd><dt>`isPagedResource()`</dt><dd>returns true is resource uri contains page info</dd><dt>`getPageNum()`</dt><dd>returns the current page num</dd><dt>`getPageSize()`</dt><dd>returns the size of the page</dd><dt>`firstPageUri()`</dt><dd>returns the Resource uri for first Single page resource</dd><dt>`nextPageUri()`</dt><dd>returns the Resource uri for next page resource or the current page resource uri</dd><dt>`prevPageUri()`</dt><dd>returns the Resource uri for next page resource or the first page resource uri</dd><dt>`hasNextPage()`</dt><dd>returns true if a next page is available. By default returns true. It can be affected by `declareActualPageSize()`
          method</dd><dt>`isLastPage()`</dt><dd>returns true if the current is the last page. By default returns false. It can be affected by `declareActualPageSize()`
          method</dd><dt>`declareActualPageSize($count)`</dt><dd>inform context of the number of item in current Page, by default is 0\. This affects `hasNextPage()`
          and `isLastPageMethods()`.</dd></dl>

Here is how `declaretActualPageSize()` affects `hasNextPage()` and `isLastPage()`:

## License

 Copyright © 2016 by  Enrico Fagnoni at [LinkedData.Center](http://LinkedData.Center/)®

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.


  