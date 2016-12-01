
#Elgin Traffic Disruptions Data API

Welcome to the public repo for Elgin Traffic Disruptions Data API.
Here you can find the specification of the data in form of XSD and WSDL schemas. 

The following files and directories are included:

*Elgin Traffic Disruptions Data API - Data Elements v20x (cust).xlsx*
- Excel file containing a definition of each field 

*elgin-api.wsdl*
- SOAP contract and definition

*elgin-api-single.wsdl*
- SOAP contract and definition for use in Microsoft Visual Studio

*xsd* <directory> - included API super types
*gml* <directory> - included by elginTypes.xsd
*samples* <directory> 
- contain data samples in both XML and Json
- Illustrates WGS84 and British National Grid coordinate systems


##WSDL
SOAP implementations 

###Java
To create Java POJOs from the WSDL and included XSD files run the following (choose any package name):
~~~~
wsimport -keep -verbose .\elgin-api.wsdl -p com.elgin.elginws.ties
~~~~

###.NET
To create C# POCOs from the WSDL and included XSD files run the following:
~~~~
svcutil.exe .\elgin-api.wsdl .\xsd\elginItems.xsd .\xsd\elginTypes.xsd .\xsd\gml\03-105r1_GML_3.1.1\GML-3.1.1\base\geometryBasic0d1d.xsd  .\xsd\gml\03-105r1_GML_3.1.1\GML-3.1.1\base\basicTypes.xsd .\xsd\gml\03-105r1_GML_3.1.1\GML-3.1.1\base\defaultStyle.xsd .\xsd\gml\03-105r1_GML_3.1.1\GML-3.1.1\base\gmlBase.xsd .\xsd\gml\03-105r1_GML_3.1.1\GML-3.1.1\xlink\xlinks.xsd .\xsd\gml\03-105r1_GML_3.1.1\GML-3.1.1\base\geometryBasic2d.xsd .\xsd\gml\03-105r1_GML_3.1.1\GML-3.1.1\base\measures.xsd .\xsd\gml\03-105r1_GML_3.1.1\GML-3.1.1\smil\smil20.xsd  .\xsd\gml\03-105r1_GML_3.1.1\GML-3.1.1\smil\smil20-language.xsd .\xsd\gml\03-105r1_GML_3.1.1\GML-3.1.1\smil\xml-mod.xsd  /language:C# /dataContractOnly /importxmltypes /out:ElginApi.ContractOnly.cs
~~~~
This will create a file including the web service contract and configuration file with the default endpoint (change this).

The above code is a bit nasty to run. This is due to the WSDL's use of <imports> to include data types in xsd files - this is perfectly normal. 

An easier option than the above is using the `elgin-api-single.wsdl` file that contains all the types and elements as the `elgin-api.wsdl` file and its includes.  However, this single file is an interpretation of the original files and has many of the types and elements copied or abbreviated from the included XSD schemas. I would use the  `elgin-api-single.wsdl` if you are not sure what you are doing in Visual Studio - make sure you change the web service endpoint in the App.config file to bind to a real web service on the Elgin side. You can use this (https://raw.githubusercontent.com/roadworks/api/master/elgin-api-single.wsdl "link") in Visual Studio.

The .NET configuration file, `App.Config`, after running the above process will look something like this.
~~~~
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
    <startup> 
        <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5.2" />
    </startup>
    <system.serviceModel>
        <bindings>
            <basicHttpBinding>
                <binding name="ElginSOAP" />
            </basicHttpBinding>
        </bindings>
        <client>
            <endpoint address="http://localhost:8080/soap/api/" binding="basicHttpBinding"
                bindingConfiguration="ElginSOAP" contract="ServiceReference.Elgin"
                name="ElginSOAP" />
        </client>
    </system.serviceModel>
</configuration>
~~~~
This is some code how you can use the client:
~~~~
using ConsoleApplication1.ServiceReference;
 
namespace ConsoleApplication1
{
    class Program
    {
        static void Main(string[] args)
        {
            using (var client = new ElginClient("ElginSOAP", "http://api.roadworks.org/XXXXXXX/"))
            {
                var param = new DataParameterType
                {
                    DataGrouping = DataCType.Items,
                    Account = "username",
                    Key = "xxxxxxxxxxxxx",
                    EntitySourceID = "3450"
                };
                var output = client.GetData(param);
            }
        }
    }
}
~~~~
###HTTP POST
The following demonstrates an example POST to the web service. Note that the XXXX denote a specific customer endpoint.
~~~~
POST http://api.roadworks.org/XXXXXXXX/ HTTP/1.1
Content-Type: text/xml; charset=utf-8
SOAPAction: "http://www.elgin.org.uk/elgin/api/GetData"
Host: api.roadworks.org
Accept-Encoding: gzip, deflate
Connection: Keep-Alive
 
<s:Envelope xmlns:s="http://schemas.xmlsoap.org/soap/envelope/">
    <s:Body xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:xsd="http://www.w3.org/2001/XMLSchema">
        <GetDataRequest xmlns="http://www.elgin.org.uk/schemas/api">
            <DataParamter DataGrouping="Items" Account="username"
                Key="xxxxxxxxxxxxx">
                <ItemType>Road closure</ItemType>
            </DataParamter>
        </GetDataRequest>
    </s:Body>
</s:Envelope>
~~~~

##Good Luck!



