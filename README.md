# UTK Exodus :flight_departure:

## About

This application is a complete rewrite of the code used to migrate UTK content from Islandora 7 to Hyku.

Unlike the previous code, this aims to be more flexible, easier to understand, and easier to use as a whole.

## Installing

To install for use, ideally use `pipx`:

```shell
pipx install utk_exodus
```

This will install the application in a virtual environment and make it available to you where ever you are in your
path, so that you can use it from anywhere without needing to understand the intricacies of Python. 

If you don't want to use `pipx`, you can install the whole library with the following command but do so mindfully:

```shell
pip install utk_exodus
```

## Before You Start

Before you start, you need to have a few things in place:

1. Exodus assumes you have the following environmental variables set appropriately:
    * `FEDORA_USER`: this is a user with read access to the Fedora repository
    * `FEDORA_PASSWORD`: the password for that Fedora user
    * `FEDORA_URI`: the base URI for where Fedora is installed
2. If you're looking for these values, you can find them in the Exodus `Environment` of this repository in `Settings`.

## Using

There are several interfaces for the application.

If you want to get works and files, and you have metadata files, use:

```shell
exodus works_and_files --path /path/to/metadata -o /path/to/directory/to/store/files
```

If you want to get works and files, and you don't have metadata files, you need to specify
a collection and a work type:

```shell
exodus works_and_files --collection "namespace:identifier" --model book -o /path/to/output/directory
```

If you just want works, use:

```shell
exodus works --path /path/to/metadata
```

If for some reason you need to create a files sheet for  works after the fact, use:

```shell
exodus add_files --sheet path/to/sheet.csv --files_sheet path/to/files_sheet.csv 
```

## What's Missing Here Right Now

* The ability to create pcdm:Collection objects.
* The ability to create a new metadata import from a previous import

## Understanding Configs

Exodus migrates works and filesets according to [the UTK Metadata mapping](https://utk-mods-to-rdf.readthedocs.io/en/latest/contents/5_technical_metadata_properties.html#mapping).
To do this, Exodus uses `yml` files for migration.  By default, exodus treats everything agnostically and relies on the 
`xpaths` section of the base mapping to determine how a concept is mapped. If a property (or properties) have complex 
rules, a class can be written to handle the special case.  When this happens, the `yml` should have a `special` 
property, and it should be defined in `MetadataMapping().__lookup_special_property()`.

An agnostic property should look like this in the `yml`:

```yml
  - name: table_of_contents
    xpaths:
      - 'mods:tableOfContents'
    property: "http://purl.org/dc/terms/tableOfContents"
```

A complex property might look like this:

```yml
  - name: title_and_alternative_title
    xpaths:
      - 'mods:titleInfo[not(@supplied)]/mods:title'
      - 'mods:titleInfo[@supplied="yes"]/mods:title'
    properties:
      - "http://purl.org/dc/terms/title"
      - "http://purl.org/dc/terms/alternative"
    special: "TitleProperty"
```

An agnostic property must always have the `property` property while a complex property may have `property` or 
`properties`.
