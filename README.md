[![Build Status](https://secure.travis-ci.org/ddeboer/DdeboerDataImportBundle.png)](http://travis-ci.org/ddeboer/DdeboerDataImportBundle)

Ddeboer Data Import Bundle
==========================

Introduction
------------
This Symfony2 bundle offers a way to import data from, and store data to, a
range of formats and media. During import, you can filter and convert all data.

Installation
------------

This library is available on [Packagist](http://packagist.org/packages/ddeboer/data-import-bundle).

To install it, add the following to your `composer.json`:

```
"require": {
    ...
    "ddeboer/data-import-bundle": "dev-master",
    ...
}
```

And run `$ php composer.phar install`.

Then add the bundle to `app/AppKernel.php`:

```
public function registerBundles()
{
    return array(
        ...
        new Ddeboer\DataImportBundle\DdeboerDataImportBundle(),
        ...
    );
}
```

Usage
-----

1. Create an `splFileObject` from the source data. You can also use a `source`
   object to retrieve this `splFileObject`: construct a source and add `source
   filters`, if you like.
2. Construct a `reader` object and pass an `splFileObject` to it.
3. Construct a `workflow` object and pass the reader to it. Add at least one
   `writer` object to this workflow. You can also add `filters` and `converters`
   to the workflow.
4. Process the workflow: this will read the data from the reader, filter and
   convert the data, and write it to the writer(s).

An example:

```
use Ddeboer\DataImportBundle\Workflow;
use Ddeboer\DataImportBundle\Source\Http;
use Ddeboer\DataImportBundle\Source\Filter\Unzip;
use Ddeboer\DataImportBundle\Reader\CsvReader;
use Ddeboer\DataImportBundle\Converter\DateTimeConverter;

(...)

// Create the source; here we use an HTTP one
$source = new Http('http://www.opta.nl/download/registers/nummers_csv.zip');

// As the source file is zipped, we add an unzip filter
$source->addFilter(new Unzip('nummers.csv'));

// Retrieve the \SplFileObject from the source
$file = $source->getFile();

// Create and configure the reader
$csvReader = new CsvReader($file);
$csvReader->setHeaderRowNumber(0);

// Create the workflow
$workflow = new Workflow($csvReader);
$dateTimeConverter = new DateTimeConverter();

// Add converters to the workflow
$workflow
    ->addConverter('twn_datumbeschikking', $dateTimeConverter)
    ->addConverter('twn_datumeind', $dateTimeConverter)
    ->addConverter('datummutatie', $dateTimeConverter)

// You can also add closures as converters
    ->addConverter('twn_nummertm',
        new \Ddeboer\DataImportBundle\Converter\CallbackConverter(
            function($input) {
                return str_replace('-', '', $input);
            }
        )
    ->addConverter('twn_nummervan',
        new \Ddeboer\DataImportBundle\Converter\CallbackConverter(
            function($input) {
                return str_replace('-', '', $input);
            }
        )

// Use one of the writers supplied with this bundle, implement your own, or use
// a closure:
    ->addWriter(new \Ddeboer\DataImportBundle\Writer\CallbackWriter(
        function($csvLine) {
            var_dump($csvLine);
        }
    );

// Process the workflow
$workflow->process();
```