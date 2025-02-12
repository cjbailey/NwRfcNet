# .NET client library for SAP NetWeawer RFC
An easy way of making SAP RFC calls from .NET. Libray is supported in Windows, Linux and macOS.

## Supported Platforms & Prerequisites

* Requires .NET Framework ( [.NET Standard 2.0](https://docs.microsoft.com/en-us/dotnet/standard/net-standard) or higher ) 
  -  .NET Framework 4.6.1 or higher
  -  .NET Core 2.0 or higher

* OS versions
  - Windows x64.
  - Redhat Linux 7 or other Linux distribution that is supported simultaneously by .NET Core and SAP NetWeaver RFC. 
  - macOS 10.12+.

* SAP NetWeaver RFC Library 7.50 SDK C++ binaries must be installed locally. For download and installation instructions check [SAP Note 2573790](https://launchpad.support.sap.com/#/notes/2573790)


## Using this package

Add the package using the `dotnet` cli:

```
$ dotnet add package NwRfcNet
```

Create a class to match SAP RFC parameters

```C#
    public class BapiCompanyOutputParameters
    {
        public CompanyDetails[] Details { get; set; }
    }

    public class CompanyDetails
    {
        public string CompanyCode { get; set; }

        public string Name { get; set; }
    }
```

Map RFC mapameters to class 

```C#
    RfcMapper mapper = new RfcMapper();

    mapper.Parameter<BapiCompanyOutputParameters>().Property(x => x.Details)
        .HasParameterName("COMPANYCODE_LIST")
        .HasParameterType(RfcFieldType.Table);

    mapper.Parameter<CompanyDetails>().Property(x => x.CompanyCode)
        .HasParameterName("COMP_CODE")
        .MaxLength(4)
        .HasParameterType(RfcFieldType.Char);

    mapper.Parameter<CompanyDetails>().Property(x => x.Name)
        .HasParameterName("COMP_NAME")
        .MaxLength(25)
        .HasParameterType(RfcFieldType.Char);
```

Open a connection to server and invoke a BAPI 

```C#
    using (var conn = new RfcConnection(userName, password, hostname, client))
    {
        conn.Open();
        using(var func = _conn.CallRfcFunction("BAPI_COMPANYCODE_GETLIST"))
        {
            func.Invoke();
        }
    }
```

Get result and display to Console

```C#
    var returnValue =  func.GetOutputParameters<BapiCompanyOutputParameters>();
    Console.WriteLine(String.Format("|{0,-20}|{1,-10}", "Company Code", "Company Name"));
    foreach (var row in returnValue.Details)
    {
        Console.WriteLine(String.Format("|{0,-20}|{1,-10}", row.CompanyCode, row.Name));
    }
```

Output should be 

| Company Code  | Company Name |
| ------------- | -------------|
| C001          | company 1    |
| C002          | Company 2    |

## Samples

Included samples in project

* List FI Comapnies
* List FI Customers
* Get Details of a FI Customer
* FI General Ledger Account

## Single sign-on (SSO)

Alternatively it's possible to sign-on with SSO instead of user and password. 

Example :  Opening a RFC connection using NTLM.  
 
```C#
    var conn = new RfcConnection(
        hostname: "server_name", 
        client: "000", 
        sncPartnername: @"p:DOMAIN\SRVACC",
        sncLib: @"gx64ntlm.dll" )
``` 

Define the SNC parameters related to your environment. If SncLib is not defined, NetWeaver RFC Library uses library defined in environment variable SNC_LIB or SNC_LIB_64.

```C#
        public string SncQop { get ; set; }

        public string SncMyname { get; set; }

        public string SncPartnername { get; set; }

        public string SncLib { get; set; }
``` 
