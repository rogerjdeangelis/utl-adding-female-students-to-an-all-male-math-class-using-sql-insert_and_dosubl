# utl-adding-female-students-to-an-all-male-math-class-using-sql-insert_and_dosubl
Adding female students to an all male math class using sql insert.
    Adding female students to an all male math class using sql insert

      Two Solutions (could be useful when dealing with a foreign sql relational database)

         1. Iterative Dosubl one insert at a time
         2. Create file of all inserts then dosubl all the inserts at once


    github
    https://tinyurl.com/yyswkzm8
    https://github.com/rogerjdeangelis/utl-adding-female-students-to-an-all-male-math-class-using-sql-insert_and_dosubl

    StackOverflow
    https://tinyurl.com/y38nunaw
    https://stackoverflow.com/questions/54642330/proc-sql-doing-a-insert-into-within-a-call-execute-in-sas


    INPUT
    =====

    data males;
      set sashelp.class(where=(sex="M") keep=name sex age obs=4);
    run;quit;

    WORK.MALES total obs=4

       NAME      SEX    AGE

      Alfred      M      14
      Henry       M      14
      James       M      12
      Jeffrey     M      13

    * ad these to the males;
    data females;
      set sashelp.class(where=(sex="F")  keep=name sex age obs=4);
    run;quit;

    WORK.FEMALES total obs=4

      NAME       SEX    AGE

      Alice       F      13
      Barbara     F      13
      Carol       F      14
      Jane        F      12

    RULES
    =====
     create these inserts from the female dataset and insert them into the male table

     values("Alice","F"13)
     values("Barbara","F"13)
     values("Carol","F"14)
     values("Jane","F"12)


    EXAMPLE OUTPUT
    ---------------

     MALES total obs=8

      NAME       SEX    AGE

      Alfred      M      14
      Henry       M      14
      James       M      12
      Jeffrey     M      13

      Alice       F      13  Added using sql insert
      Barbara     F      13
      Carol       F      14
      Jane        F      12

    WORK.LOG total obs=4

              INSERT                STATUS

      values("Alice","F"13)      Insert Worked
      values("Barbara","F"13)    Insert Worked
      values("Carol","F"14)      Insert Worked
      values("Jane","F"12)       Insert Worked



    PROCESS
    =======

    -----------------------------------------
    1. Iterative Dosubl one insert at a time
    -----------------------------------------

    * just in case;

    proc datasets lib=work;
    delete males females;
    run;quit;

    * make data;
    data males;
      set sashelp.class(where=(sex="M") keep=name sex age obs=4);
    run;quit;

    data females;
      set sashelp.class(where=(sex="F")  keep=name sex age obs=4);
    run;quit;

    * SOLUTION;

    data log;

      set females;
      insert=cats('values("',name,'","',sex,'"',age,')');
      call symputx('insert',insert);
      rc=dosubl('
        proc sql;
         insert into males &insert
        ;quit;
        %let cc=&syserr;
      ');

      if symgetn('cc')=0 then status="Insert Worked";
      else status="Insert Failed";
      keep insert status;

    ru;quit;

    ------------------------------------------------------------------
    2. Create file of all inserts then dosubl all the inserts at once
    ------------------------------------------------------------------

    * PREP;
    filename ghost clear; * just in case;

    proc datasets lib=work;
    delete males females;
    run;quit;

    data males;
      set sashelp.class(where=(sex="M") keep=name sex age obs=4);
    run;quit;

    data females;
      set sashelp.class(where=(sex="F")  keep=name sex age obs=4);
    run;quit;

    * SOLUTION;
    filename ghost temp;
    data log;

       if _n_=0 then do;%let rc=%sysfunc(dosubl(
          data _null_;
            set females;
            file ghost;
            if _n_=1 then put "proc sql;insert into males ";
            insert=cats('values("',name,'","',sex,'",',age,')');
            put insert;
            putlog insert;
          run;quit;
          ));
        end;

       rc=dosubl('
         %include ghost;
        ;quit;
        %let cc=&syserr;
      ');

      if symgetn('cc')=0 then status="All Insert Worked";
      else status="Insert Failed";
      keep status;
    run;quit;

    WORK.LOG total obs=1

           STATUS

      All Insert Worked



