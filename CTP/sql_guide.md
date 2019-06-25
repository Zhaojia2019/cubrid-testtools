# 1 Test Objective
SQL is an ANSI (American National Standards Institute) standard computer language, but there are still many different versions of SQL.However, in order to be compatible with ANSI standards, they must all support some major commands (such as SELECT, UPDATE, DELETE, INSERT, WHERE, and so on) in a similar way.  
CUBRID SQL detection, SQL detection for each build to detect the basic functions of the database and ensure the correct output of commands.In addition, SQL is a category of code coverage tests.  

# 2 Tools Introduction and environment construction
To facilitate the execution of various test requirements, CUBRID QA has developed a range of automated tools. This chapter describes the environment construction.

This is an implementation of SQL functions that we used to test CUBRID. 

## 2.1 Linux
**Test Machines**  

No|role|user|ip|hostname
:---:|:--:|:---:|:---:|:---:
1|Test node|sql1|192.168.1.76|func01
2|Test node|sql2|192.168.1.76|func01
3|Test node|sql3|192.168.1.76|func01
4|Test node|sqlbycci|192.168.1.76|func01
5|Test node|sql|192.168.1.77|func02

* Install CTP  
    * Step 1: Checkout git repository
    ```
    cd ~
    git clone https://github.com/CUBRID/cubrid-testtools.git
    cd ~/cubrid-testtools 
    git checkout develop
    cp -rf CTP ~/
    ```
    * Step 2: Configurations
      * touch and configure ~/CTP/conf/common.conf
      ```
      git_user=cubridqa
      git_pwd=PASSWORD
      git_email=dl_cubridqa_bj_internal@navercorp.com
      default_ssh_pwd=PASSWORD
      default_ssh_port=22

      grepo_service_url=rmi://192.168.1.91:11099
      coverage_controller_pwd=PASSWORD

      qahome_db_driver=cubrid.jdbc.driver.CUBRIDDriver
      qahome_db_url=jdbc:cubrid:192.168.1.86:33080:qaresu:dba::
      qahome_db_user=dba
      qahome_db_pwd=

      qahome_server_host=192.168.1.86
      qahome_server_port=22
      qahome_server_user=qahome
      qahome_server_pwd=PASSWORD

      activemq_user=admin
      activemq_pwd=PASSWORD
      activemq_url=failover:tcp://192.168.1.91:61616?wireFormat.maxInactivityDurationInitalDelay=30000

      mail_from_nickname=CUBRIDQA_BJ
      mail_from_address=dl_cubridqa_bj_internal@navercorp.com
      ```
      * configure ~/.bash_profile
      ```
      export CTP_HOME=$HOME/CTP
      export CTP_SKIP_UPDATE=0
      export CTP_BRANCH_NAME=develop
      . ~/.cubrid.sh
      export PATH=$HOME/CTP/bin:$HOME/CTP/common/script:$PATH:$HOME/.local/bin:$PATH:$HOME/bin

      export TZ='Asia/Seoul'
      export LC_ALL=en_US
      ```
      source ~/.bash_profile

      * touch and configure ~/CTP/conf/sql_local.conf
      ```
      #Copyright (c) 2016, Search Solution Corporation. All rights reserved.
      #--------------------------------------------------------------------
      #
      #Redistribution and use in source and binary forms, with or without 
      #modification, are permitted provided that the following conditions are met:
      #
      #  * Redistributions of source code must retain the above copyright notice, 
      #    this list of conditions and the following disclaimer.
      #
      #  * Redistributions in binary form must reproduce the above copyright 
      #    notice, this list of conditions and the following disclaimer in 
      #    the documentation and/or other materials provided with the distribution.
      #
      #  * Neither the name of the copyright holder nor the names of its contributors may be used to endorse or promote products 
      #    derived from this software without specific prior written permission.
      #
      #THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, 
      #INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE 
      #DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, 
      #SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR 
      #SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, 
      #WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE 
      #USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

      # SQL section - a section for CTP tool configuration when executing sql/medium testing
      [sql]
      # The location of your testing scenario
      scenario=${HOME}/cubrid-testcases/sql
      # Configure an alias name for testing result
      test_category=sql
      # Config file for I18N client charset configuration and init session parameter via 'set system parameter xxx'
      jdbc_config_file=test_default.xml
      # Config database charset for db creation
      db_charset=en_US
      # If test need do make locale or not
      need_make_locale=yes

      # SQL cubrid.conf section - a section for cubrid.conf configuration
      [sql/cubrid.conf]
      # To decide if the Java store procedure will be used when testing
      java_stored_procedure=yes
      # Allow scenario to change database system parameter
      test_mode=yes
      # To increase the speed of execution
      max_plan_cache_entries=1000
      # To increase the speed of execution
      unicode_input_normalization=no
      # To change port of cubrid_port_id to avoid port conflict
      cubrid_port_id=1285
      # In order to simulate the scenario customer use
      ha_mode=yes
      # To reduce the lock wait time to fast testing execution
      lock_timeout=10sec

      # SQL cubrid_ha.conf section - a section for ha related configuration
      [sql/cubrid_ha.conf]
      # Once ha_mode=yes is configured in cubrid.conf, you will require to configure cubrid_ha.conf except ha_db_list 
      ha_mode=yes
      # To reduce memory use
      ha_apply_max_mem_size=300
      # To set what port will be used for ha_port_id
      ha_port_id=12859

      # SQL cubrid_broker.conf query editor section - a section to change parameters under query_editor
      [sql/cubrid_broker.conf/%query_editor]
      # To close one service to avoid port conflict and reduce configuration complexity
      SERVICE=OFF

      # SQL cubrid_broker.conf broker1 section - a section to change parameters under broker1
      [sql/cubrid_broker.conf/%BROKER1]
      # To change broker port to avoid port conflict, if you are sure the port will not conflict, just ignore.
      BROKER_PORT=33285
      # To change ID of shared memory used by CAS, if you are sure the port will not conflict, just ignore.
      APPL_SERVER_SHM_ID=33285

      # SQL cubrid_broker.conf broker section - a section to configure parameters under broker section
      [sql/cubrid_broker.conf/broker]
      # To change the identifier of shared memory to avoid conflict to cause server start fail
      MASTER_SHM_ID=32851
      ```
      * touch and configure ~/CTP/conf/sql_by_cci_template.conf
      ```
      #Copyright (c) 2016, Search Solution Corporation. All rights reserved.
      #--------------------------------------------------------------------
      #
      #Redistribution and use in source and binary forms, with or without 
      #modification, are permitted provided that the following conditions are met:
      #
      #  * Redistributions of source code must retain the above copyright notice, 
      #    this list of conditions and the following disclaimer.
      #
      #  * Redistributions in binary form must reproduce the above copyright 
      #    notice, this list of conditions and the following disclaimer in 
      #    the documentation and/or other materials provided with the distribution.
      #
      #  * Neither the name of the copyright holder nor the names of its contributors may be used to endorse or promote products 
      #    derived from this software without specific prior written permission.
      #
      #THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, 
      #INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE 
      #DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, 
      #SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR 
      #SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, 
      #WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE 
      #USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

      # SQL section - a section for CTP tool configuration when executing sql/medium testing
      [sql]
      # The location of your testing scenario
      scenario=${HOME}/cubrid-testcases/sql
      sql_interface_type=cci
      # Configure an alias name for testing result
      test_category=sql

      # Configuration file for jdbc connection initialization, just ignore it when executing sql_by_cci
      jdbc_config_file=test_default.xml
      # Config database charset for db creation
      db_charset=en_US
      # If test need do make locale or not
      need_make_locale=yes

      # SQL cubrid.conf section - a section for cubrid.conf configuration
      [sql/cubrid.conf]
      # To decide if the Java store procedure will be used when testing
      java_stored_procedure=yes
      # Allow scenario to change database system parameter
      test_mode=yes
      # To increase the speed of execution
      max_plan_cache_entries=1000
      # To increase the speed of execution
      unicode_input_normalization=no
      # To change port of cubrid_port_id to avoid port conflict
      cubrid_port_id=1285
      # In order to simulate the scenario customer use
      ha_mode=yes
      # To reduce the lock wait time to fast testing execution
      lock_timeout=10sec

      # SQL cubrid_ha.conf section - a section for ha related configuration
      [sql/cubrid_ha.conf]
      # Once ha_mode=yes is configured in cubrid.conf, you will require to configure cubrid_ha.conf except ha_db_list 
      ha_mode=yes
      # To reduce memory use
      ha_apply_max_mem_size=300
      # To set what port will be used for ha_port_id
      ha_port_id=12859

      # SQL cubrid_broker.conf query editor section - a section to change parameters under query_editor
      [sql/cubrid_broker.conf/%query_editor]
      # To close one service to avoid port conflict and reduce configuration complexity
      SERVICE=OFF

      # SQL cubrid_broker.conf broker1 section - a section to change parameters under broker1
      [sql/cubrid_broker.conf/%BROKER1]
      # To change broker port to avoid port conflict, if you are sure the port will not conflict, just ignore.
      BROKER_PORT=33285
      # To change ID of shared memory used by CAS, if you are sure the port will not conflict, just ignore.
      APPL_SERVER_SHM_ID=33285

      # SQL cubrid_broker.conf broker section - a section to configure parameters under broker section
      [sql/cubrid_broker.conf/broker]
      # To change the identifier of shared memory to avoid conflict to cause server start fail
      MASTER_SHM_ID=32851
      ```
    * Step 3: Make sure that CTP installed successfully.
    ```
    $ ctp.sh -h
    Welcome to use CUBRID Test Program (CTP)
    usage: ctp.sh <sql|medium|shell|ha_repl|isolation|jdbc|unittest> -c
                  <config_file>
     -c,--config <arg>   provide a configuration file
     -h,--help           show help
        --interactive    interactive mode to run single test case or cases in
                         a folder
     -v,--version        show version

    utility: ctp.sh webconsole <start|stop>

    For example: 
            ctp.sh sql -c conf/sql.conf
            ctp.sh medium -c conf/medium.conf
            ctp.sh shell -c conf/shell.conf
            ctp.sh ha_repl -c conf/ha_repl.conf
            ctp.sh isolation -c conf/isolation.conf
            ctp.sh jdbc -c conf/jdbc.conf
            ctp.sh sql              #use default configuration file: /home/memory1/CTP/conf/sql.conf
            ctp.sh medium           #use default configuration file: /home/memory1/CTP/conf/medium.conf
            ctp.sh shell            #use default configuration file: /home/memory1/CTP/conf/shell.conf
            ctp.sh ha_repl          #use default configuration file: /home/memory1/CTP/conf/ha_repl.conf
            ctp.sh isolation                #use default configuration file: /home/memory1/CTP/conf/isolation.conf
            ctp.sh jdbc             #use default configuration file: /home/memory1/CTP/conf/jdbc.conf
            ctp.sh unittest #use default configuration file: /home/memory1/CTP/conf/unittest.conf
            ctp.sh sql medium       #run both sql and medium with default configuration
            ctp.sh medium medium    #execute medium twice
            ctp.sh webconsole start #start web console to view sql test results
    ```
* Check out test cases
  ```
  cd ~
  git clone https://github.com/CUBRID/cubrid-testcases.git 
  ```
* Install CUBRID.
* touch start_test.sh
```
stop_consumer.sh 

prefix=`date "+%Y%m%d%H%M%S"`
cp nohup.out nohup.out.$prefix
echo "" > nohup.out

nohup start_consumer.sh -q [QUEUE_NAME] -exec [run_name] &
```
* vi ~/.autoUpdate.sh
```
#!/bin/bash
export HOME=/home/[user]
export USER=[user]
if [ -f ~/.bash_profile ];
then
          . ~/.bash_profile
fi
set -x
cd $HOME/CTP/common/sched/../script
chmod u+x *
./stop_consumer.sh
./upgrade.sh
nohup $HOME/CTP/common/script/start_consumer.sh -q [QUEUE_NAME] -exec [run_name]  2>&1 >> $HOME/nohup.out &
cd -
```
 * sh start_test.sh
## 2.2 Windows
**Test Machines**  

No|role|user|ip|hostname
:---:|:--:|:---:|:---:|:---:
1|Test node|qa|192.168.1.161|winfunc01

**Deploy worker node**  
**Prepare**  
create and set users  
We need create a new user for dailyqa test: qa  
passwd is the normal root passwd.  
1) add user qa  
2) use 'Change Account Type' to change user 'qa' to 'Administrator'  

**Deploy**  
* Login as user 'qa'  

    1. install JDK  
    jdk version must greater than 1.6.0_07  
    We use jdk-8u201-windows-x64  

    2. Install visual studio 2017  
    Visual studio is used by make_local.bat  
    When install visual studio 2017, Choose 'Workloads' view(tab), in 'Windows (3)' section, Choose "Desktop development with C++", then click 'Install' or 'Modify' to start the installation.  
    After installation, check system variable '%VS140COMNTOOLS%'  
        1) If 'VS140COMNTOOLS' is not add to the system variables automatically, please add it manually.  
        Variable name: VS140COMNTOOLS  
        Variable value: C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\Common7\Tools\   
        2) If 'VS140COMNTOOLS' is add to the system variables automatically, please check its value. Sometimes, the value is not correct for make_locale to use it. In this situation, please change it to the correct one.  
        e.g.  
        Wrong: C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\Tools\  
        Correct: C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\Common7\Tools\ (required for last '\')  

    3. install cygwin  
        1) We need choose this packages manually since they will not be installed by default: wget, zip, unzip, dos2unix, bc, expect.  
        gcc and mingw packages do not need to be installed.  
        2) check the versions of these packages (or components): gawk, grep, sed  
        The invalid versions for cygwin components:  
        grep: 3.0-2  
        gawk: 4.1.4-3  
        sed: 4.4-1  
        We must use the versions before the versions list above.  
        In my test, I use:  
        gawk: 4.1.3-1  
        grep: 3.0-1  
        sed: 4.2.2-3  

      To install the old versions. please refer to this comment Install old packages of cygwin  
    
    4. install git  
    https://git-for-windows.github.io/  
    In the installation wizard, choose these options:  
    'Adjusting your PATH environment', choose 'Use Git from the Windows Command Prompt'  
    'Confifuring the line ending conversions', choose 'Checkout as-is, commit as-is'  

    5. install a text editor tool which can used both in linux and windows format  
    I intall 'Notepad++ v7.5.1'  

    6. install cubrid  
    Use the msi installation file to install cubrid for the first time.  

    7. set 'Environment Variables'   
      1) After install cubrid by msi file, these system parameter will be added automatically:  
      ```
      CUBRID
      C:\CUBRID\

      CUBRID_DATABASES
      C:\CUBRID\databases
      ```
      2) Add new 'System variables':  
      ```
      JAVA_HOME
      C:\Program Files\Java\jdk1.8.0_201

      CTP_BRANCH_NAME
      develop

      CTP_SKIP_UPDATE
      0
      ```
      3) Edit 'path'  
      add '%JAVA_HOME%\bin C:\cygwin64\bin' in the 'path'.  

    8. Install CTP  
      1)Please follow the guide to install CTP.  
      2) touch ~/CTP/conf/sql_local.conf  
      Here is the config file that we use for current daily QA test: sql_local.conf  

    9. Check out test cases  
    ```
    cd ~
    git clone https://github.com/CUBRID/cubrid-testcases.git 
    ```
    10. touch start_test.sh    
    ```
    start_test.sh
    rm nohup.out
    nohup start_consumer.sh -q QUEUE_CUBRID_QA_SQL_WIN64 -exec run_sql &
    ```
    11. vi ~/.autoUpdate.sh  
    ```
    autoUpdate.sh
    #!/bin/bash 
    export HOME=/cygdrive/c
    export USER=
    if [ -f ~/.bash_profile ]; 
    then 
        . ~/.bash_profile 
    fi 
    set -x 
    cd /cygdrive/c/CTP/common/sched/../script 
    chmod u+x *
    ./stop_consumer.sh
    ./upgrade.sh
    nohup /cygdrive/c/CTP/common/script/start_consumer.sh -q QUEUE_CUBRID_QA_SQL_WIN64 -exec run_sql 2>&1 >> /cygdrive/c/nohup.out &
    cd -
    ```
    sh start_test.sh  
# 3 Test run  
## 3.1 manual test  
### ctp  
* run ctp.sh
```
ctp.sh sql -c CTP/conf/sql_local.conf --interactive
```
Modify the working directory and configuration parameters in the file CTP/conf/sql_local.conf\
* run test
```
======================================  Welcome to Interactive Mode ======================================  
Usage: 
    help         print the usage of interactive
    run <arg>    the path of case file to run
    run_cci <arg>    the path of case file to run_cci by cci driver
    quit         quit interactive mode 

For example:
    run .                                                 #run the current directory cases
    run ./_001_primary_key/_002_uniq                      #run the cases which are based on the relative path
    run test.sql                                          #run the test.sql file
    run /home/user1/dailyqa/trunk/scenario/sql/_02_object #run the cases which are based on the absolute path
    run_cci ./_001_primary_key/_002_uniq                  #run the cases which are based on the relative path
    run_cci ./_001_primary_key/_002_uniq/test.sql         #run the cases file
sql>run .
```
图片 ![文字]（./doc/media/image1.png）
### csql  
* cubrid createdb testdb en_US  
* cubrid server start testdb
* csql -u dba testdb
图片 ![文字]（./doc/media/image1.png）
图片 ![文字]（./doc/media/image1.png）
## 3.2 automatic monitoring configuration  
**Test Machines**  

No|role|user|ip|hostname|QUEUE_NAME|run_name
:---:|:--:|:---:|:---:|:---:|:---:|:---:
1|Test node|sql1|192.168.1.76|func01|QUEUE_CUBRID_QA_SQL_LINUX_GIT|run_sql
2|Test node|sql2|192.168.1.76|func01|QUEUE_CUBRID_QA_SQL_LINUX_GIT|run_sql
3|Test node|sql3|192.168.1.76|func01|QUEUE_CUBRID_QA_SQL_LINUX_GIT|run_sql
4|Test node|sqlbycci|192.168.1.76|func01|QUEUE_CUBRID_QA_SQL_CCI_LINUX_GIT|run_sql_by_cci
5|Test node|sql|192.168.1.77|func02|QUEUE_CUBRID_QA_SQL_PERF_LINUX|run_sql
6|Test node|qa|192.168.1.161|winfunc01|QUEUE_CUBRID_QA_SQL_WIN64|run_sql

* Enable configuration listening  
  * touch start_test.sh  
  ```
  stop_consumer.sh 

  prefix=`date "+%Y%m%d%H%M%S"`
  cp nohup.out nohup.out.$prefix
  echo "" > nohup.out

  nohup start_consumer.sh -q [QUEUE_NAME] -exec [run_name] &
  ```
  * vi ~/.autoUpdate.sh  
  ```
  #!/bin/bash
  export HOME=/home/[user]
  export USER=[user]
  if [ -f ~/.bash_profile ];
  then
            . ~/.bash_profile
  fi
  set -x
  cd $HOME/CTP/common/sched/../script
  chmod u+x *
  ./stop_consumer.sh
  ./upgrade.sh
  nohup $HOME/CTP/common/script/start_consumer.sh -q [QUEUE_NAME] -exec [run_name]  2>&1 >> $HOME/nohup.out &
  cd -
  ```
   * sh start_test.sh  
* send test message  
  login message@192.168.1.91 and send test message like:  
  ```
  sender.sh  [QUEUE_NAME]  [CI_BUILD] [Category] default
  ```
  [QUEUE_NAME]:Message queue name  
  [CI_BUILD]:Corresponds to the build installation package address
  [Category]:sql,sql_debug,sql_by_cci,medium,medium_debug  
# 4. Verify test Results  
How to view qahome results:[image]  
**Validation step**  
* Step 1: bug recurrence  
Use CTP and CSQL tools to reproduce the test
* Step 2: analyze the cause of the bug  
  **Several cases**  
  1. Modify CUBRID caused.  
  Check whether CUBRID version differences and modifications will cause case execution answer changes.
  2. Unstable environment.  
  For some failures caused by excessive command execution time or network reasons, the case needs to be perfected.
  3. It's not what you expected  
  This is where you need to create a bug instead of the expected result.
