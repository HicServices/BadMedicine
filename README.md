# BadMedicine

[![Build Status](https://github.com/HICServices/BadMedicine/actions/workflows/testpack.yml/badge.svg?branch=develop)](https://travis-ci.org/HicServices/BadMedicine) [![NuGet Badge](https://buildstats.info/nuget/HIC.BadMedicine)](https://www.nuget.org/packages/HIC.BadMedicine/)

Library and CLI for randomly generating medical data like you might get out of an Electronic Health Records (EHR) system.  It is intended for generating data for demos and testing ETL / cohort generation/ data management tools.

BadMedicine differs from other random data generators e.g. Mockaroo, SQL Data Generator etc in that data generated is based on (simple) models generated from live EHR datasets collected for over 30 years in Tayside and Fife (UK).  This makes the data generated recognisable (codes used, frequency of codes etc) from a clinical perspective and representative of the problems (ontology mapping etc) that data analysts would encounter working with real medical data.

Datasets generated are not suitable for training AI algorithms etc (See [What is Modelled?](#what-is-modelled))

## Datasets

The following synthetic datasets can be produced.

| Dataset        | Description           |
| ------------- |:-------------:|
| Demography      | Address and patient details as might appear in the CHI register |
| Biochemistry      | Lab test codes as might appear in Sci Store lab system extracts |
| Prescribing      | Prescription data of prescribed drugs |
| Carotid Artery Scan      | Scan results for Carotid Artery |
| Hospital Admissions | ICD9 and ICD10 codes for admission to hospital |
| Maternity | Records of births etc |

## Usage:

BadMedicine is available as a [nuget package](https://www.nuget.org/packages/HIC.BadMedicine/) for linking as a library

The standalone CLI (BadMedicine.exe) is available in the [releases section of Github](https://github.com/HicServices/BadMedicine/releases)

Usage is as follows:

```
BadMedicine.exe c:\temp\
```

You can change how much data is produced (e.g. 500 patients, 10000 records per dataset):

```
BadMedicine.exe c:\temp\ 500 10000
```

Or run only a single dataset:

```
BadMedicine.exe c:\omg 5000 200000 -l -d CarotidArteryScan
```

You can seed the generator (Guids generated will still differ)

```
BadMedicine.exe c:\omg 5000 200000 -l -d CarotidArteryScan -s 5000
```

## Building

Building requires MSBuild 15 or later (or Visual Studio 2017 or later).  You will also need to install the DotNetCore 2.2 SDK.

You can build a OS specific binary

First build BadMedicine.csproj
```
dotnet publish BadMedicine.csproj -r win-x64 --self-contained
cd .\bin\Debug\netcoreapp2.2\win-x64\
```
## Direct to Database

You can generate data directly into a relational database (instead of onto disk).

To turn this mode on rename the file `BadMedicine.template.yaml` to `BadMedicine.yaml` and provide the connection strings to your database e.g.:

```yaml
Database:
  # Set to true to drop and recreate tables described in the Template
  DropTables: false
  # The connection string to your database
  ConnectionString: server=(localdb)\MSSQLLocalDB;Integrated Security=true;
  # Your DBMS provider ('MySql', 'PostgreSql','Oracle' or 'MicrosoftSQLServer')
  DatabaseType: MicrosoftSQLServer
  # Database to create/use on the server
  DatabaseName: BadMedicineTestData
```

## Library Usage

You can generate test data for your program yourself by referencing the [nuget package](https://www.nuget.org/packages/HIC.BadMedicine/):

```csharp
//Seed the random generator if you want to always produce the same randomisation
var r = new Random(100);

//Create a new person
var person = new Person(r);

//Create test data for that person
var a = new HospitalAdmissionsRecord(person,person.DateOfBirth,r);

Assert.IsNotNull(a.Person.CHI);
Assert.IsNotNull(a.Person.DateOfBirth);
Assert.IsNotNull(a.Person.Address.Line1);
Assert.IsNotNull(a.Person.Address.Postcode);
Assert.IsNotNull(a.AdmissionDate);
Assert.IsNotNull(a.DischargeDate);
Assert.IsNotNull(a.Condition1);
```

## What is Modelled?

Data generated by BadMedicine is driven by Aggregate distributions of real health data collected in Tayside (UK).  This means that codes appear in data with the frequency that match real data.  For example in the Hospital Admissions data we can see that ICD9 codes (denoted by dash) cease being recorded in ~1997 in favour of ICD10 codes and we can see the most common admission conditions are sensible:

![alt text](./Images/MainConditionDistribution.png)

*ICD 9 and ICD 10 codes in Condition1 (the main condition) upon Hospital Admission*

## What is not Modelled?

No inter dataset / inter record level randomisation model exists.  For example the following would **not** be modelled:

- If a patient is on Drug A they are more likely to also be on Drug B
- Hospitalisations are more likely to be at the beginning/end of a patients life
- Drug A is likely to be given to patients discharged having been treated for condition Y
