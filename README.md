# utl_Using_the_DOW_to_add_concatenated_values_to_every_observations_by_group
Using the DOW to add concatenated values to every observations by group. Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverfl SAS community.
    Using the DOW to add concatenated values to every observations by group

    github
    https://goo.gl/sm3FHj
    https://github.com/rogerjdeangelis/utl_Using_the_DOW_to_add_concatenated_values_to_every_observations_by_group

    Stackoverflow R
    https://stackoverflow.com/questions/49029068/recode-identifier-for-different-combinations

      Two Solutions

       1. SAS/WPS same results
       2. WPS/PROC R


    INPUT
    =====

     RULES
       Concatenate group with values separated by '_'

     WORK.HAVE total obs=14     RULES ( add this column)

       COMBI  GROUP  VALUE |    COMBI2
                           |
         A      x      1   |    x_1_2_3_4
         A      x      2   |    x_1_2_3_4
         A      x      3   |    x_1_2_3_4
         A      x      4   |    x_1_2_3_4

         B      x      1   |    x_1_3
         B      x      3   |    x_1_3

         C      x      2   |
         C      x      3   |
         D      y      1   |
         D      y      2   |
         E      y      1   |
         E      y      3   |
         F      y      2   |
         F      y      3   |


    PROCESS
    =======

     1. SAS/WPS

        data want;
          length combi2 $200 ;
          retain combi2;

         * rollup value and prefix with group value (x or y);
         do until (last.combi);
           set have;
           by combi notsorted;
           combi2=catx('_',combi2,value);
           if last.combi then combi2=cats(group,'_',combi2);
         end;

         * apply the retained concatenated values to each ob;
         do until (last.combi);
           set have;
           by combi notsorted;
           output;
         end;

         * reset for next group;
         combi2='';

        run;quit;

     2. R  (working code

        want<-have %>%
            group_by(GROUP, COMBI) %>%
            arrange(GROUP, COMBI, VALUE) %>%
            mutate(COMBI2 = paste(GROUP, paste0(VALUE, collapse = "_"), sep = "_"));


    OUTPUT
    ======

     WORK.WANTWPS total obs=14

         GROUP    COMBI    VALUE    COMBI2

           x        A        1      x_1_2_3_4
           x        A        2      x_1_2_3_4
           x        A        3      x_1_2_3_4
           x        A        4      x_1_2_3_4
           x        B        1      x_1_3
           x        B        3      x_1_3
           x        C        2      x_2_3
           x        C        3      x_2_3
           y        D        1      y_1_2
           y        D        2      y_1_2
           y        E        1      y_1_3
           y        E        3      y_1_3
           y        F        2      y_2_3
           y        F        3      y_2_3

    *                _               _       _
     _ __ ___   __ _| | _____     __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \   / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/  | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|   \__,_|\__,_|\__\__,_|

    ;

    options validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.have;
     input Group$ Combi$ Value$;
    cards4;
    x A 1
    x A 2
    x A 3
    x A 4
    x B 1
    x B 3
    x C 2
    x C 3
    y D 1
    y D 2
    y E 1
    y E 3
    y F 2
    y F 3
    ;;;;
    run;quit;

    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __
    / __|/ _ \| | | | | __| |/ _ \| '_ \
    \__ \ (_) | | |_| | |_| | (_) | | | |
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|

    ;

     * SAS;

     data want;
       length combi2 $200 ;
       retain combi2;

      * rollup value and prefix with group value (x or y);
      do until (last.combi);
        set have;
        by combi notsorted;
        combi2=catx('_',combi2,value);
        if last.combi then combi2=cats(group,'_',combi2);
      end;

      * apply the retained concatenated values to each ob;
      do until (last.combi);
        set have;
        by combi notsorted;
        output;
      end;

      * reset for next group;
      combi2='';

     run;quit;

    * WPS;

    %utl_submit_wps64('

    libname wrk "%sysfunc(pathname(work))";
    libname sd1 "d:/sd1";

     data wrk.want;

      length combi2 $200 ;
      retain combi2;

      do until (last.combi);
        set sd1.have;
        by combi notsorted;
        combi2=catx("_",combi2,value);
        if last.combi then combi2=cats(group,"_",combi2);
      end;

      * apply the retained concatenated values to each ob;
      do until (last.combi);
        set sd1.have;
        by combi notsorted;
        output;
      end;

      * reset for next group;
      combi2="";

    run;submit;

    ');

     WPS/PROC R

     %utl_submit_wps64('
     libname sd1 "d:/sd1";
     options set=R_HOME "C:/Program Files/R/R-3.3.2";
     libname wrk "%sysfunc(pathname(work))";
     libname hlp "C:\Program Files\SASHome\SASFoundation\9.4\core\sashelp";
     proc r;
     submit;
     source("C:/Program Files/R/R-3.3.2/etc/Rprofile.site", echo=T);
     library(haven);
     have<-read_sas("d:/sd1/have.sas7bdat");
     require(dplyr);
     want<-have %>%
         group_by(GROUP, COMBI) %>%
         arrange(GROUP, COMBI, VALUE) %>%
         mutate(COMBI2 = paste(GROUP, paste0(VALUE, collapse = "_"), sep = "_"));
     endsubmit;
     import r=want data=wrk.wantwps;
     run;quit;
     ');

