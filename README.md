Salesforce DX Essentials
========================

[![Version](https://img.shields.io/npm/v/sfdx-essentials.svg)](https://npmjs.org/package/sfdx-essentials)
[![Downloads/week](https://img.shields.io/npm/dw/sfdx-essentials.svg)](https://npmjs.org/package/sfdx-essentials) 
[![License](https://img.shields.io/npm/l/sfdx-essentials.svg)](https://github.com/nvuillam/sfdx-essentials/blob/master/package.json) 
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](http://makeapullrequest.com)

# PLUGIN

Toolbox for Salesforce DX to provide some very helpful additional features to base sfdx commands.

**SFDX Essentials** commands are focused on DevOps, but not only (cleaning, sources consistency check ...)

Easy to integrate in a CD/CI process (Jenkins Pipeline,CircleCI...)

Contributions are ery welcome, please run **npm run lint:fix** before making a new PR

Command list

| Command | Description |
| ------------- | ------------- |
| [essentials:change-dependency-version](#essentialschange-dependency-version) | **Replace other managed packages dependency version number** ( very useful when you build a managed package over another managed package, like Financial Services Cloud ) |
| [essentials:check-sfdx-project-consistency](#essentialscheck-sfdx-project-consistency) | **Check consistency between a SFDX project files and package.xml files** |
| [essentials:filter-metadatas](#essentialsfilter-metadatas) | **Filter metadatas generated from a SFDX Project** in order to be able to deploy only part of them on an org |
| [essentials:filter-xml-content](#essentialsfilter-xml-content) | **Filter content of metadatas (XML)** in order to be able to deploy only part of them on an org |
| [essentials:fix-lightning-attributes-names](#essentialsfix-lightning-attributes-names) | **Replace reserved lightning attribute names in lightning components and apex classes** ( if you named a lightning attribute like a custom apex class, since Summer 18 you simply can not generate a managed package again) |
| [essentials:generate-permission-sets](#essentialsgenerate-permission-sets) | **Generate permission sets** from packageXml file depending JSON configuration file |
| [essentials:migrate-object-model](#essentialsmigrate-object-model) | **Migrate sources from an object model to a new object model** |
| [essentials:order-package-xml](#essentialsorder-package-xml) | **Reorder alphabetically the content of package.xml file(s)** |
| [essentials:uncomment](#essentialsuncomment) | **Uncomment lines in sfdx/md files** (useful to manage @Deprecated annotations with managed packages) |

Please contribute :)

# INSTALLATION

```
    sfdx plugins:install sfdx-essentials
```

- Windows users: [sfdx plugin generator](https://github.com/forcedotcom/sfdx-plugin-generate) is bugged on windows (hardcode call of linux rm instruction) , so you may use [Git Bash](https://gitforwindows.org/) to run this code ( at least while it installs the plugin dependencies )

- CI Users: As the plugin is not signed, to install it from a Dockerfile or a script:
```
    echo 'y' | sfdx plugins:install
```

# UPGRADE

Its seems that sfdx plugins:update and sfdx update does not always work, in that case , uninstall then reinstall the plugin
```
    sfdx plugins:uninstall sfdx-essentials
    sfdx plugins:install sfdx-essentials
```

# CONTRIBUTE

- Fork the repo and clone it on your computer
- run $ sfdx plugins:link
- Now your calls to sfdx essentials are performed on your local sources 
- Once your code is ready and documented, please make a pull request :)

# COMMANDS

## `essentials:filter-metadatas`

Allows to filter metadatas folder generated by sfdx force:source:convert , using your own package.xml file

This can help if you need to deploy only part of the result of sfdx force:source:convert into a org, by filtering the result (usually in mdapi_output_dir) to keep only the items referenced in your own package.xml file

WARNING: This version does not support all the metadata types yet, please contribute if you are in a hurry :)

```
USAGE
  $ sfdx essentials:filter-metadatas OPTIONS

OPTIONS
  -i, --inputfolder=inputfolder    Input folder (default: "." )
  -o, --outputfolder=outputfolder  Output folder (default: filteredMetadatas)
  -p, --packagexml=packagexml      package.xml file path

DESCRIPTION
  
     Package.xml types currently managed:

     - ApexClass
     - ApexComponent
     - ApexPage
     - ApexTrigger
     - ApprovalProcess
     - AuraDefinitionBundle
     - AuthProvider
     - BusinessProcess
     - ContentAsset
     - CustomApplication
     - CustomField
     - CustomLabel
     - CustomMetadata
     - CustomObject
     - CustomObjectTranslation
     - CustomSite
     - CustomTab
     - Document
     - EmailTemplate
     - EscalationRules
     - FlexiPage
     - Flow
     - FieldSet
     - GlobalValueSet
     - GlobalValueSetTranslation
     - HomePageLayout
     - ListView
     - LightningComponentBundle
     - Layout
     - NamedCredential
     - Network
     - NetworkBranding
     - PermissionSet
     - Profile
     - Queue
     - QuickAction
     - RecordType
     - RemoteSiteSetting
     - Report
     - SiteDotCom
     - StandardValueSet
     - StandardValueSetTranslation
     - StaticResource
     - Translations
     - ValidationRule
     - WebLink
     - Workflow

```

_See [conversion tables](https://github.com/nvuillam/sfdx-essentials/blob/master/src/common/metadata-utils.ts)_

EXAMPLES

```
  $ sfdx essentials:filter-metadatas -p myPackage.xml

  $ sfdx essentials:filter-metadatas -i md_api_output_dir -p myPackage.xml -o md_api_filtered_output_dir

  $ sfdx force:source:convert -d tmp/deployDemoQuali/
  $ sfdx essentials:filter-metadatas -i tmp/deployDemoQuali/ -p myPackage.xml -o tmp/deployDemoQualiFiltered/
  $ sfdx force:mdapi:deploy -d tmp/deployDemoQualiFiltered/ -w 60 -u DemoQuali

```

_See code: [src/commands/essentials/filter-metadatas.ts](https://github.com/nvuillam/sfdx-essentials/blob/master/src/commands/essentials/filter-metadatas.ts)_

## `essentials:filter-xml-content`

When you perform deployments from one org to another, the features activated in the target org may not fit the content of the sfdx/metadata files extracted from the source org.

You may need to filter some elements in the XML files, for example in the Profiles

This script requires a filter-config.json file following this example

```
USAGE
  $ sfdx essentials:filter-xml-content OPTIONS

OPTIONS
  -i, --inputfolder=inputfolder    Input folder (default: "." )
  -o, --outputfolder=outputfolder  Output folder (default: Input Folder + _xml_content_filtered)
  -p, --configFile=configFile      JSON file containing configuration. Default: filter-config.json

EXAMPLE
  $ sfdx essentials:filter-xml-content -i "retrieveUnpackaged" 

```

_See JSON configuration example: [examples/filter-xml-content-config.json](https://github.com/nvuillam/sfdx-essentials/blob/master/examples/filter-xml-content-config.json)_

_See code: [src/commands/essentials/filter-xml-content.ts](https://github.com/nvuillam/sfdx-essentials/blob/master/src/commands/essentials/filter-xml-content.ts)_

## `essentials:change-dependency-version`

Allows to change an external package dependency version

Can also :

- remove package dependencies if --namespace and --remove arguments are sent 
- update API version

```
USAGE
  $ sfdx essentials:change-dependency-version OPTIONS

OPTIONS
  -f, --folder=folder              SFDX project folder containing files
  -j, --majorversion=majorversion  Major version
  -m, --minorversion=minorversion  Minor version
  -n, --namespace=namespace        Namespace of the managed package
  -a, --apiversion=apiversion      API Version
  -r, --remove                     Remove package dependency

EXAMPLES
  $ sfdx essentials:change-dependency-version -n FinServ -j 214 -m 7

  $ sfdx essentials:change-dependency-version -n FinServ -r

  $ sfdx essentials:change-dependency-version -a 47.0
```

_See code: [src/commands/essentials/change-dependency-version.ts](https://github.com/nvuillam/sfdx-essentials/blob/master/src/commands/essentials/change-dependency-version.ts)_

## `essentials:fix-lightning-attributes-names`

If you named a lightning attribute like a custom apex class, since Summer 18 you simply can not generate a managed package again.

This command lists all custom apex classes and custom objects names , then replaces all their references in lightning components and also in apex classes with their camelCase version.

Ex : MyClass_x attribute would be renamed myClassX

```
USAGE
  $ sfdx essentials:fix-lightning-attributes-names OPTIONS

OPTIONS
  -f, --folder=folder              SFDX project folder containing files (usually 'force-app/main/default'). Default : '.'

EXAMPLE
  $ sfdx essentials:fix-lightning-attributes-names 
```

_See code: [src/commands/essentials/fix-lightning-attributes-names.ts](https://github.com/nvuillam/sfdx-essentials/blob/master/src/commands/essentials/fix-lightning-attributes-names.ts)_

## `essentials:uncomment`

Once you flagged a packaged method as **@Deprecated** , you can not deploy it in an org not used for generating a managed package

This commands allows to uncomment desired lines just before making a deployment

Before :

``` 
// @Deprecated SFDX_ESSENTIALS_UNCOMMENT
global static List<OrgDebugOption__c> setDebugOption() {
	return null;
}
```

After :

```
@Deprecated // Uncommented by sfdx essentials:uncomment (https://github.com/nvuillam/sfdx-essentials)
global static List<OrgDebugOption__c> setDebugOption() {
	return null;
}
```

```
USAGE
  $ sfdx essentials:uncomment OPTIONS

OPTIONS
  -f, --folder=folder              SFDX project folder containing files (usually 'force-app/main/default'). Default : '.'
  -k, --uncommentKey=someString              Uncomment key. Default : 'SFDX_ESSENTIALS_UNCOMMENT'


EXAMPLE
  $ sfdx essentials:uncomment --folder "./Projects/DevRootSource/tmp/deployPackagingDxcDevFiltered" --uncommentKey "SFDX_ESSENTIALS_UNCOMMENT_DxcDev_"
```

_See code: [src/commands/essentials/uncomment.ts](https://github.com/nvuillam/sfdx-essentials/blob/master/src/commands/essentials/uncomment.ts)_

## `essentials:check-sfdx-project-consistency`

Allows to compare the content of a SFDX and the content of one or several package.xml files ( append, if several )

```
USAGE
  $ sfdx essentials:check-sfdx-project-consistency OPTIONS

OPTIONS
  -p, --packageXmlList=someString                   List of package.xml files path
  -i, --inputfolder=someString                      SFDX Project folder . Default : '.'
  -d, --ignoreDuplicateTypes=someString             List of types to ignore while checking for duplicates in package.xml files
  -f, --failIfError                                 Script failing if errors are founds. Default: false
  -c, --chatty                                      Chatty logs. Default: false
  -j, --jsonLogging                                 JSON logs. Default: false                              

EXAMPLE
  $  sfdx essentials:check-sfdx-project-consistency -p "./Config/packageXml/package_DevRoot_Managed.xml,./Config/packageXml/package_DevRoot_xDemo.xml" -i "./Projects/DevRootSource/force-app/main/default" -d "Document,EmailTemplate" --failIfError
```
_See log example: [src/commands/essentials/examples/check-sfdx-project-consistency.log](https://github.com/nvuillam/sfdx-essentials/blob/master/examples/check-sfdx-project-consistency.log)_

_See code: [src/commands/essentials/check-sfdx-project-consistency.ts](https://github.com/nvuillam/sfdx-essentials/blob/master/src/commands/essentials/check-sfdx-project-consistency.ts)_

![Check SFDX project consistency log image](https://github.com/nvuillam/sfdx-essentials/blob/master/examples/check-sfdx-project-consistencys-log.png "Check SFDX project consistency log image")

## `essentials:generate-permission-sets`

Allows to generate permission sets in XML format used for SFDX project from package.xml file depending on JSON configuration file 

```
USAGE
  $ sfdx essentials:generate-permission-sets OPTIONS

OPTIONS
  -c, --configFile=configFile              JSON configuration file that will be used in order to generate permission sets
  -p, --packageXml=someString              package.xml file that will be used in order to generate permission sets
  -f, --sfdxSourcesFolder=someString       SFDX Sources folder (used to filter required and masterDetail fields)
  -s, --nameSuffix=someString              If provided, suffix string appended to file name, label and description of generated PS
  -o, --outputfolder=someString            Output folder for PS files (default: '.')
  -v, --verbose                            Display more logs           

EXAMPLES
  $  sfdx essentials:generate-permission-sets -c "./Config/generate-permission-sets.json" -p "./Config/packageXml/package_DevRoot_Managed.xml" -f "./Projects/DevRootSource/force-app/main/default" -o "./Projects/DevRootSource/force-app/main/default/permissionsets"

  $  sfdx essentials:generate-permission-sets -c "./Config/generate-permission-sets.json" -p "./Config/packageXml/package_DevRoot_xDemo.xml" -f "./Projects/DevRootSource/force-app/main/default" --nameSuffix Custom -o "./Projects/DevRootSource/force-app/main/default/permissionsets"
```

_See JSON configuration example: [examples/generate-permission-sets-config.json](https://github.com/nvuillam/sfdx-essentials/blob/master/examples/generate-permission-sets-config.json)_

_See log example: [src/commands/essentials/examples/generate-permission-sets.log](https://github.com/nvuillam/sfdx-essentials/blob/master/examples/generate-permission-sets.log)_

_See code: [src/commands/essentials/generate-permission-sets.ts](https://github.com/nvuillam/sfdx-essentials/blob/master/src/commands/essentials/generate-permission-sets.ts)_

![Generate permission sets log image](https://github.com/nvuillam/sfdx-essentials/blob/master/examples/generate-permission-sets-log.png "Generate permission sets log image")

## `essentials:migrate-object-model`

Use this command if you need to replace a SObject by another one in all your sfdx sources

```
USAGE
  $ sfdx essentials:migrate-object-model OPTIONS

OPTIONS
  -c, --configFile=configFile              JSON configuration file 
  -i, --inputFolder=someString              Input folder (default: "." )
  -f, --fetchExpressionList     Fetch expression list. Let default if you dont know. ex: /aura/**/*.js,./aura/**/*.cmp,./classes/*.cls,./objects/*/fields/*.xml,./objects/*/recordTypes/*.xml,./triggers/*.trigger,./permissionsets/*.xml,./profiles/*.xml,./staticresources/*.json'
  -r, --replaceExpressions       Replace expressions using fetchExpressionList. default: true
  -d, --deleteFiles     Delete files with deprecated references. default: true
  -k, --deleteFilesExpr     Delete files matching expression. default: true
  -s, --copySfdxProjectFolder   Copy sfdx project files after process. default: true
  -v, --verbose   Verbose

EXAMPLES
  $ sfdx essentials:migrate-object-model -c "./config/migrate-object-model-config.json"

  $ sfdx essentials:migrate-object-model -c migration/config-to-oem.json -i Config/packageXml --fetchExpressionList "./package*.xml" --no-deleteFiles --no-deleteFilesExpr --no-copySfdxProjectFolder
```

_See JSON configuration example: [examples/migrate-object-model-config.json](https://github.com/nvuillam/sfdx-essentials/blob/master/examples/migrate-object-model-config.json)_

_See code: [src/commands/essentials/migrate-object-model.ts](https://github.com/nvuillam/sfdx-essentials/blob/master/src/commands/essentials/migrate-object-model.ts)_

## `essentials:order-package-xml`

Developers have the bad habit to input package.xml files in a non-alphabetical order. 

Use this command to reorder alphabetically your package.xml files !

```
USAGE
  $ sfdx essentials:order-package-xml OPTIONS

OPTIONS
  -p, --packagexml=folderOrFile                Path to package.xml file, or folder containing package.xml files ( *.xml are processed)

EXAMPLES
  $ sfdx essentials:order-package-xml -p "./Config/packageXml/package.xml"

  $ sfdx essentials:order-package-xml -p "./Config/packageXml"
```

_See code: [src/commands/essentials/order-package-xml.ts](https://github.com/nvuillam/sfdx-essentials/blob/master/src/commands/essentials/order-package-xml.ts)_
