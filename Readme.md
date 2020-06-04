# DSMobile CI/CD Python script
*Automate the security analysis of mobile Android applications using [DerScanner Mobile](https://mobile.derscanner.com/).*

This script is designed to integrate mobile applications security analysis in the continuous development process (CI / CD).
During the execution of the script, the application is sent to the DerScanner Mobile system for analysis. The output is a json file with detailed results.

## Launch options
Currently, several launch options are supported:
 * Local apk file
 * Applications from [HockeyApp](https://hockeyapp.net/)
 * Applications from [AppCenter](https://appcenter.ms)

## Launch parameters
The launch options depend on the location of the apk file sent for analysis. Also, there are required parameters that must be specified for any type of launch:
 * `dsmobile_url` - network address for DS Mobile (the path to the root without the final /), when using the cloud version - `https://saas.mobile.derscanner.com`
 * `profile` - ID of the profile to be analyzed
 * `testcase` - ID of the test case to be executed; it is possible to run several test cases by specifying their IDs separated with spaces
 * `token` - CI/CD access token (refer to our documentation for ways to retrieve the token)
 * `distribution_system` - distribution method for the application; possible values: `file`, `hockeyapp` or `appcenter`. For detailed information refer to the respective sections below.

### Optional parameters:
 * `report` - type of the report to be generated upon completion of the analysis. Possible values:
 * `separate` - an individual scan report. This is the default report type for a single test case ID. When selected for multiple test case IDs creates an individual report for each test case
 * `standard` - the default report type for several test case IDs, consolidates duplicated defects found during different scans into one (IDs remain unchanged)
 * `grouping` - a report that groups defects of the same type and provides full details for the defects

 You can select more than one value to create several necessary reports, for example: "report" "separate" "grouping" "standard". For scans with one test case, separate and grouping reports can be created. 

### Local launch
This type of launch implies that the application apk file is located locally.
To select this method at startup, you must specify the parameter `distribution_system file`. In this case, the required parameter must specify the path to the file: `file_path`

### HockeyApp
To download an application from the HockeyApp distribution system you need to select the `distribution_system = hockeyapp` parameter. Also, you need to specify the following mandatory parameters:

 * `hockey_token` (mandatory parameter) - API access token. Look in the [HockeyApp documentation](https://rink.hockeyapp.net/manage/auth_tokens) how to retrieve it.
 * `hockey_version` (optional parameter) - this parameter downloads the specific version of the application in accordance with its version ID (the `version` field in the [API](https://support.hockeyapp.net/kb/api/api-versions)). If this parameter is not set, the latest available version of the application ("latest") will be downloaded.
 * `hockey_bundle_id` or `hockey_public_id` (mandatory parameter)
    * `hockey_bundle_id` - ID of Android application or, alternatively, the package name (`com.dsmobile.app.example`). This option launches search among all HockeyApp applications and thereafter picks an application with the corresponding ID. API field - [bundle_identifier](https://support.hockeyapp.net/kb/api/api-apps).
    * `hockey_public_id` - ID of an application inside the HockeyApp system. This parameter downloads an application with the corresponding ID. API field - [public_identifier](https://support.hockeyapp.net/kb/api/api-apps)

### AppCenter
To download application from AppCenter distribution system you need to select the `distribution_system appcenter` parameter. Also, you need to specify the following mandatory parameters:
 * `appcenter_token` - API access token. Look in official documentation to [learn how to retrieve it]((https://docs.microsoft.com/en-us/appcenter/api-docs/)).
 * `appcenter_owner_name` - owner of the application. Look in official documentation to learn how to retreive the [owner name](https://docs.microsoft.com/en-us/appcenter/api-docs/#find-your-app-center-app-name-and-owner-name).
 * `appcenter_app_name` - the name of the application in the AppCenter system. Look in official documentation to [learn how to retrieve it](https://docs.microsoft.com/en-us/appcenter/api-docs/#find-your-app-center-app-name-and-owner-name)
 * `appcenter_release_id` or `appcenter_app_version`
    * `appcenter_release_id` - ID of the specific release of the application to be downloaded from AppCenter. There is a possibility to select the "latest" value - the [latest available version](https://openapi.appcenter.ms/#/distribute/releases_getLatestByUser) of the application will be downloaded.
    * `appcenter_app_version` - this parameter finds and downloads the specific version of the application in accordance with its version ID (shown in Android Manifest) (the "version" field in the [AppCenter Documentation](https://openapi.appcenter.ms/#/distribute/releases_list))

## Launch examples

#### Local file
To start local file analysis:

```
python3.6 run-dsmobile-scan.py --distribution_system file --file_path "/dsmobile/demo/apk/dsmobile-demo.apk" --dsmobile_url "https://saas.mobile.derscanner.com" --profile 1 --testcase 4 --token "eyJ0eXA4OiJKA1QiLbJhcGciO5JIU4I1NiJ1.eyJzdaJqZWNcX2lkIj53LCJle5AiOjf1OTM5OTU3MjB1.hfI6c4VN_U2mo5VfRoENPvJCvpxhLzjHqI0gxqgr2Bs"
```

As a result, an automated analysis of the `dsmobile-demo.apk` application with a profile with `id 1` and a test case with `id 4` will be launched.

#### HockeyApp by bundle_identifier and version
To run application analysis from a HockeyApp system:

```
python3.6 run-dsmobile-scan.py --distribution_system hockeyapp --hockey_token 18bc81146d374ba4b1182ed65e0b3aaa --bundle_id com.dsmobilesecurity.demo --hockey_version 31337 --dsmobile_url "https://saas.mobile.derscanner.com" --profile 2 --testcase 3 --token "eyJ0eXA4OiJKA1QiLbJhcGciO5JIU4I1NiJ1.eyJzdaJqZWNcX2lkIj53LCJle5AiOjf1OTM5OTU3MjB1.hfI6c4VN_U2mo5VfRoENPvJCvpxhLzjHqI0gxqgr2Bs"
```

As a result, an application with the package ID `com.dsmobilesecurity.demo` and version` 31337` will be found on the HockeyApp system. It will be downloaded and an automated analysis will be performed for it with a profile with `id 2` and a test case with` id 3`.

#### HockeyApp with public identifier and the latest available version
To start scannig the latest version of an application in HockeyApp system using the application's public ID:

```
python3.6 run-dsmobile-scan.py --distribution_system hockeyapp --hockey_token 18bc81146d374ba4b1182ed65e0b3aaa --public_id "1234567890abcdef1234567890abcdef" --dsmobile_url "https://saas.mobile.derscanner.com" --profile 2 --testcase 3 --token "eyJ0eXA4OiJKA1QiLbJhcGciO5JIU4I1NiJ1.eyJzdaJqZWNcX2lkIj53LCJle5AiOjf1OTM5OTU3MjB1.hfI6c4VN_U2mo5VfRoENPvJCvpxhLzjHqI0gxqgr2Bs"
```
As a result, the latest available version of the application with the unique public ID `1234567890abcdef1234567890abcdef` will be found in HockeyApp system. The application will be downloaded and automatically analyzed using the profile with `ID 2` and the test case with `ID 3`.

#### AppCenter with the release ID
To start scannig an application using its name, the name of the owner and the release ID, the following command should be entered:

```
python3.6 run-dsmobile-scan.py --distribution_system appcenter --appcenter_token 18bc81146d374ba4b1182ed65e0b3aaa --appcenter_owner_name test_org_or_user --appcenter_app_name dsmobile_debug_version_of_test --appcenter_release_id 710 --dsmobile_url "https://saas.mobile.derscanner.com" --profile 2 --testcase 3 --token "eyJ0eXA4OiJKA1QiLbJhcGciO5JIU4I1NiJ1.eyJzdaJqZWNcX2lkIj53LCJle5AiOjf1OTM5OTU3MjB1.hfI6c4VN_U2mo5VfRoENPvJCvpxhLzjHqI0gxqgr2Bs"
```
As a result, the `debug_version_of_test` application with release `ID 710` will be found among applications of the specified owner (user or organization `test_org_or_user`). This version of the release will be downloaded and sent to DS Mobile for security analysis.

To download the latest version of the release you need to use the following parameter: `appcenter_release_id latest`. The command line will look as follows:

```
python3.6 run-dsmobile-scan.py --distribution_system appcenter --appcenter_token 18bc81146d374ba4b1182ed65e0b3aaa --appcenter_owner_name "test_org_or_user" --appcenter_app_name "dsmobile_debug_version_of_test" --appcenter_release_id latest --dsmobile_url "https://saas.mobile.derscanner.com" --profile 2 --testcase 3 --token "eyJ0eXA4OiJKA1QiLbJhcGciO5JIU4I1NiJ1.eyJzdaJqZWNcX2lkIj53LCJle5AiOjf1OTM5OTU3MjB1.hfI6c4VN_U2mo5VfRoENPvJCvpxhLzjHqI0gxqgr2Bs"
```

As a result, the latest available release of the application will be downloaded.

#### AppCenter by application version
To start the analysis of the application by the known name, owner and version (`version_code` in` Android Manifest`), you need to run the following command:

```
python3.6 run-dsmobile-scan.py --distribution_system appcenter --appcenter_token 18bc81146d374ba4b1182ed65e0b3aaa --appcenter_owner_name "test_org_or_user" --appcenter_app_name "dsmobile_debug_version_of_test" --appcenter_app_version 31337 --dsmobile_url "https://saas.mobile.derscanner.com" --profile 2 --testcase 3 --token "eyJ0eXA4OiJKA1QiLbJhcGciO5JIU4I1NiJ1.eyJzdaJqZWNcX2lkIj53LCJle5AiOjf1OTM5OTU3MjB1.hfI6c4VN_U2mo5VfRoENPvJCvpxhLzjHqI0gxqgr2Bs"
```

As a result, in the owner workspace (user or organization `test_org_or_user`) will be found application `dsmobile_debug_version_of_test` and will be found a release in which the version of the application `31337` was specified. This version will be downloaded and submitted for security analysis to DSMobile.
