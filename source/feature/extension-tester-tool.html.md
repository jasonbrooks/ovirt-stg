---
title: Extension tester tool
category: feature
authors: omachace
wiki_category: Feature
wiki_title: Features/Extension tester tool
wiki_revision_count: 4
wiki_last_updated: 2015-05-27
---

# Extension tester tool

Extension tester tool serves to user to be able test his configuration of his oVirt extensions. Tool provides command line interface to send specific commands to these extensions. We currently support logger and aaa extension, but there will be more in future.

#### Command-line interface

    ovirt-engine-extensions-tool
      --extension-file=[STRING]  Extension file to be used.
      --extensions-dir=[STRING]  Path to directory of extensions.
      --help  Show test tool help.
      --log-file=[STRING]  File where log will be stored.
      --log-level=[STRING]  Log level of test tool.
      --version  Show version of test tool.

### AAA module

#### Command-line interface

    aaa
      --help  Show help for aaa module.
      authz-fetch_principal_record
        --extension-name
        --user-name
      authn-authenticate_credentials
        --extension-name
        --user-name
        --password=
            Password can be specified in one of the following format:
              interactive: [default]
              pass:string
              env:envkey
              file:file
      login-user
        --user-name
        --password
            Password can be specified in one of the following format:
              interactive: [default]
              pass:string
              env:envkey
              file:file
        --profile-name
      search
        --profile-name
        --user-name
        --group-name
        --user-id
        --group-id

### Logger module

Logger module provides functionality to setup specific logging of oVirt. You can find more information [here](https://gerrit.ovirt.org/gitweb?p=ovirt-engine-extension-logger-log4j).

#### Command-line interface

    logger 
      --help  Show help for logger module.
      log-record
          --extension-name=[STRING]  Name of logger extension.
          --help  Show help for log-record action.
          --level=[STRING]  Level of log message to be sent.
          --logger-name=[STRING]  Name of logger.
          --message=[STRING]  Message which should be sent to logger.

#### Example

This example log message with text 'test' at level 'INFO' using logger name 'test-logger'. It fill find extension named 'ovirt-logger' in directory <i>/etc/ovirt-engine/extensions.d</i>

    ovirt-engine-extensions-tool logger log-record --extension-name=ovirt-logger --message=test --logger-name=test-logger --level=INFO

Author: --[Ondra Machacek](User:Omachace) ([talk](User talk:Omachace))

<Category:Feature>
