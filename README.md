    %let pgm=utl-personal-altair-slc-convert-sas-datastep-to-slc-sql-odbc-r-python;

    %stop_submission;

    Personal altair slc convert SAS DataStep to slc sql odbc r python

    PROBLEM: select first occurrence by sex where the age is 14

     I am new to the personal Altair slc but there appears to be a problem with odbc sqlite?

    Too long to post to listserves, see githib

    github
    https://tinyurl.com/ybm73xfe
    https://github.com/rogerjdeangelis/utl-personal-altair-slc-convert-sas-datastep-to-slc-sql-odbc-r-python

     The personal Altair slc supports ODBC without an access  product.
     I downloaded and installed the sqlite odbc driver, but using
     that driver failed. I posted the problem, hopefully we will get an solution.
     Note, this may be my problem not Altairs.

    github
    https://tinyurl.com/5au8cwpk
    https://github.com/rogerjdeangelis/utl-sqllite-odbc-driver-works-in-r-but-not-in-personal-altair-slc

    stackoverflow sas
    https://tinyurl.com/3vvxa5zz
    https://stackoverflow.com/questions/79766528/converting-sas-data-steps-into-proc-sql-with-row-number-partition-but-altair-wor

    CONTENTS

       1 slc datstep
       2 slc proc sql
       3 slc odbc (works)
       4 slc r sqlite
       5 slc python sqlite
       6 sqlpartitionx macro
          (Because I use monotonic use it only as is to sequence groups
          or one overall sequence Do not make it a view or add any subqueies..
          If you complicate the code beyond a simple sequencing, the
          sas optimizer might reorder the groups for faster performance
          and the patitioning will be out of order)
    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */
    &_init_;
    options
      validvarname=upcase;
    data class;
      input
        name$
        sex$ age;
    cards4;
    Barbara F 13
    Alice   F 13
    Carol   F 14
    Alfred  M 14 *  select these two
    Henry   M 14
    James   U 14 *
    Ronald  U 15
    Robert  U 12
    Thomas  U 11
    ;;;;
    run;quit;

    /*       _            _       _            _
    / |  ___| | ___    __| | __ _| |_ __ _ ___| |_ ___ _ __
    | | / __| |/ __|  / _` |/ _` | __/ _` / __| __/ _ \ `_ \
    | | \__ \ | (__  | (_| | (_| | || (_| \__ \ ||  __/ |_) |
    |_| |___/_|\___|  \__,_|\__,_|\__\__,_|___/\__\___| .__/
                                                      |_|
    */

    data want;
      set class;
      by sex;
      if first.sex;
      if age=14;
    run;quit;

    OUTPUT
    ======

     NAME     SEX    AGE

    Alfred     M      14
    James      U      14

    /*___        _                                         _
    |___ \   ___| | ___   _ __  _ __ ___   ___   ___  __ _| |
      __) | / __| |/ __| | `_ \| `__/ _ \ / __| / __|/ _` | |
     / __/  \__ \ | (__  | |_) | | | (_) | (__  \__ \ (_| | |
    |_____| |___/_|\___| | .__/|_|  \___/ \___| |___/\__, |_|
                         |_|                            |_|
    */

    &_init_;
    proc sql;
      create
         table sexone(where=(age=14)) as
      select
         name
        ,sex
        ,age
      from
         %sqlpartitionx(class,by=sex)
      where
         partition=1
    ;quit;

    proc print data=sexone;
    run;

    OUTPUT
    ======

    Altair SLC

    Obs     NAME     SEX    AGE

     1     Alfred     M      14
     2     James      U      14

    /*____       _                  _ _             __       _ _
    |___ /   ___| | ___    ___   __| | |__   ___   / _| __ _(_) |___
      |_ \  / __| |/ __|  / _ \ / _` | `_ \ / __| | |_ / _` | | / __|
     ___) | \__ \ | (__  | (_) | (_| | |_) | (__  |  _| (_| | | \__ \
    |____/  |___/_|\___|  \___/ \__,_|_.__/ \___| |_|  \__,_|_|_|___/

    */

    &_init_;
    libname mydb odbc noprompt="Driver=SQLite3 ODBC Driver;Database=d:\sqlite\new.db;";

    /*---- in case you rerun ----*/
    proc sql;
      drop table mydb.class
    ;quit;

    data mydb.class;
      set class;
    run;quit;

    proc print data=mydb.class;
    run;quit;

    libname mydb clear;

    OUTPUT
    ======

    Obs    NAME        SEX         AGE

     1     Barbara     F            13
     2     Alice       F            13
     3     Carol       F            14
     4     Alfred      M            14
     5     Henry       M            14
     6     James       U            14
     7     Ronald      U            15
     8     Robert      U            12
     9     Thomas      U            11

    /*  _         _                         _ _ _
    | || |    ___| | ___   _ __   ___  __ _| (_) |_ ___
    | || |_  / __| |/ __| | `__| / __|/ _` | | | __/ _ \
    |__   _| \__ \ | (__  | |    \__ \ (_| | | | ||  __/
       |_|   |___/_|\___| |_|    |___/\__, |_|_|\__\___|
                                         |_|
    */

    &_init_;
    proc datasets lib=work
     nodetails nolist;
     delete want;
    run;quit;

    proc r;
    export data=class r=class;
    submit;
    library(sqldf)
     want<-sqldf('
      select
         name
        ,sex
        ,age
      from
        (select
           *
          ,row_number() over (partition by sex) as partition
        from
           class )
      where
         partition=1 and age=14
      ')
    want
    endsubmit;
    import data=want r=want;
    run;quit;

    proc print data=want;
    run;quit;

    OUTPUT
    ======

    Altair SLC

        NAME SEX AGE
    1 Alfred   M  14
    2  James   U  14


    /*___        _                    _   _                            _ _ _
    | ___|   ___| | ___   _ __  _   _| |_| |__   ___  _ __   ___  __ _| (_) |_ ___
    |___ \  / __| |/ __| | `_ \| | | | __| `_ \ / _ \| `_ \ / __|/ _` | | | __/ _ \
     ___) | \__ \ | (__  | |_) | |_| | |_| | | | (_) | | | |\__ \ (_| | | | ||  __/
    |____/  |___/_|\___| | .__/ \__, |\__|_| |_|\___/|_| |_||___/\__, |_|_|\__\___|
                         |_|    |___/                               |_|

    SLC Python export/import fails with newer versions
    Python 3.13.0
    NumPy version: 2.1.3
    Pandas version: 2.2.3

    Need to down grade (not tested)
    I have too many packages using the newer version

    Your python installation is not supported by SLC 2025:
    "SLC 2025 requires python ?3.12.9, numpy ?1.26.4 and pandas ?2.2.2"
    I'd recommend downgrading python, numpy and pandas
    */

    &_init_;
    options
      validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.class;
      input
        name$
        sex$ age;
    cards4;
    Barbara F 13
    Alice   F 13
    Carol   F 14
    Alfred  M 14
    Henry   M 14
    James   U 14
    Ronald  U 15
    Robert  U 12
    Thomas  U 11
    ;;;;
    run;quit;

    proc datasets lib=sd1 nolist nodetails;
     delete pywant;
    run;quit;

    options noerrorabend;
    options set=PYTHONHOME "D:\python310";
    proc python;
    submit;
    import pyreadstat as ps
    import pandas as pd
    import pyreadr as pr
    exec(open('c:/wpsoto/fn_pythonx.py').read());
    have,meta = ps.read_sas7bdat('d:/sd1/class.sas7bdat');
    want=pdsql('''
     select
        name
       ,sex
       ,age
     from
       (select
          *
         ,row_number() over (partition by sex) as partition
       from
          have )
     where
        partition=1 and age=14
       ''')
    print(want)
    pr.write_rds('d:/rds/pywant.rds',pywant)
    endsubmit;
    run;quit;

    &_init_;
    options noerrorabend;
    options set=RHOME "D:\d451";
    %wps_py2sastable(
       inp=d:/rds/pywant.pyds
      ,out=rwant );

    proc print data=work.rwant;
    run;quit;

    OUTPUT
    ======

    Altair SLC

    The PYTHON Procedure

         NAME SEX   AGE
    0  Alfred   M  14.0
    1   James   U  14.0


    LOG
    ===

    060      ODS _ALL_ CLOSE;
    1061      FILENAME WPSWBHTM TEMP;
    NOTE: Writing HTML(WBHTML) BODY file d:\wpswrk\_TD18592\#LN00032
    1062      ODS HTML(ID=WBHTML) BODY=WPSWBHTM GPATH="d:\wpswrk\_TD18592";
    1063      &_init_;
    1064      proc datasets lib=sd1 nolist nodetails;
    1065       delete pywant;
    1066      run;quit;
    NOTE: SD1.PYWANT (memtype="DATA") was not found, and has not been deleted
    NOTE: Procedure datasets step took :
          real time : 0.000
          cpu time  : 0.000


    1067
    1068      options noerrorabend;
    1069      options set=PYTHONHOME "D:\python310";
    1070      proc python;
    1071      submit;
    1072      import pyreadstat as ps
    1073      import pandas as pd
    1074      import pyreadr as pr
    1075      exec(open('c:/wpsoto/fn_pythonx.py').read());
    1076      have,meta = ps.read_sas7bdat('d:/sd1/class.sas7bdat');
    1077      want=pdsql('''
    1078       select
    1079          name
    1080         ,sex
    1081         ,age
    1082       from
    1083         (select
    1084            *
    1085           ,row_number() over (partition by sex) as partition
    1086         from
    1087            have )
    1088       where
    1089          partition=1 and age=14
    1090         ''')
    1091      print(want)
    1092      pr.write_rds('d:/rds/pywant.rds',want)
    1093      endsubmit;

    NOTE: Submitting statements to Python:

    NOTE: <string>:16: SADeprecationWarning: The _ConnectionFairy.connection attribute is deprecated; please use 'driver_connection' (deprecated since: 2.0)


    1094      run;quit;
    NOTE: Procedure python step took :
          real time : 1.571
          cpu time  : 0.000


    1095
    1096      &_init_;
    1097      options noerrorabend;
    1098      options set=RHOME "D:\d451";
    1099      %wps_py2sastable(
    1100         inp=d:/rds/pywant.rds
    1101        ,out=rwant );

    NOTE: The infile 'c:\wpsoto\wps_py2rdataframe.sas' is:
          Filename='c:\wpsoto\wps_py2rdataframe.sas',
          Owner Name=T7610\Roger,
          File size (bytes)=984,
          Create Time=11:06:54 Sep 25 2025,
          Last Accessed=16:31:37 Sep 29 2025,
          Last Modified=14:12:29 Sep 25 2025,
          Lrecl=32767, Recfm=V

    NOTE: The file 'c:\wpsoto\wps_py2rdataframeout.sas' is:
          Filename='c:\wpsoto\wps_py2rdataframeout.sas',
          Owner Name=T7610\Roger,
          File size (bytes)=0,
          Create Time=11:28:51 Sep 25 2025,
          Last Accessed=16:33:23 Sep 29 2025,
          Last Modified=16:33:23 Sep 29 2025,
          Lrecl=32767, Recfm=V

    proc datasets lib=work
      nolist nodetails;
    delete rwant;
    run;quit;
    options set=RHOME "D:\d451";
    proc r;
    submit;
    rwant <- readRDS("d:/rds/pywant.rds")
    head(rwant)
    endsubmit;
    import data=rwant r=rwant;
    ;quit;run;
    NOTE: 12 records were read from file 'c:\wpsoto\wps_py2rdataframe.sas'
          The minimum record length was 80
          The maximum record length was 80
    NOTE: 12 records were written to file 'c:\wpsoto\wps_py2rdataframeout.sas'
          The minimum record length was 80
          The maximum record length was 94
    NOTE: The data step took :
          real time : 0.002
          cpu time  : 0.015


    Start of %INCLUDE(level 1) c:/wpsoto/wps_py2rdataframeout.sas
    1102    +  proc datasets lib=work
    1103    +    nolist nodetails;
    1104    +  delete rwant;
    1105    +  run;quit;
    NOTE: Deleting "WORK.RWANT" (memtype="DATA")
    NOTE: Procedure datasets step took :
          real time : 0.001
          cpu time  : 0.000


    1106    +  options set=RHOME "D:\d451";
    1107    +  proc r;
    1108    +  submit;
    1109    +  rwant <- readRDS("d:/rds/pywant.rds")
    1110    +  head(rwant)
    1111    +  endsubmit;
    NOTE: Using R version 4.5.1 (2025-06-13 ucrt) from d:\r451

    NOTE: Submitting statements to R:

    > rwant <- readRDS("d:/rds/pywant.rds")

    NOTE: Processing of R statements complete

    > head(rwant)
    1112    +  import data=rwant r=rwant;
    NOTE: Creating data set 'WORK.rwant' from R data frame 'rwant'
    NOTE: Data set "WORK.rwant" has 2 observation(s) and 3 variable(s)

    1113    +  ;quit;run;
    NOTE: Procedure r step took :
          real time : 0.356
          cpu time  : 0.000


    NOTE: 2 observations were read from "WORK.rwant"
    NOTE: Procedure print step took :
          real time : 0.004
          cpu time  : 0.000


    End of %INCLUDE(level 1) c:/wpsoto/wps_py2rdataframeout.sas
    1114
    1115      proc print data=work.rwant;
    1116      run;quit;
    NOTE: 2 observations were read from "WORK.rwant"
    NOTE: Procedure print step took :
          real time : 0.020
          cpu time  : 0.000


    1117
    1118      quit; run;
    1119      ODS _ALL_ CLOSE;
    1120      FILENAME WPSWBHTM CLEAR;

    /*__              _                  _   _ _   _
     / /_   ___  __ _| |_ __   __ _ _ __| |_(_) |_(_) ___  _ __ __  __
    | `_ \ / __|/ _` | | `_ \ / _` | `__| __| | __| |/ _ \| `_ \\ \/ /
    | (_) |\__ \ (_| | | |_) | (_| | |  | |_| | |_| | (_) | | | |>  <
     \___/ |___/\__, |_| .__/ \__,_|_|   \__|_|\__|_|\___/|_| |_/_/\_\
                   |_| |_|
    */

    %macro sqlpartitionx(dsn,by=team,minus=1)/
       des="Improved sqlpartition that maintains data order";
     ( select
         *
         ,max(seq) as seq
       from
         (select
           *
          ,seq-min(seq) + 1 as partition
         from
           (select *, &minus*monotonic() as seq from &dsn)
         group
           by &by )
       group
           by &by, seq
       having
           1=1)
    %mend sqlpartitionx;

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */

