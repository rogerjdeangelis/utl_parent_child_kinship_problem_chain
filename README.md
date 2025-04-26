# utl_parent_child_kinship_problem_chain
Parent child kinship problem chain


    Parent child kinship problem chain

      WPS and SAS (same results)

    Related to (but slightly different)

    see
    https://goo.gl/eL2C3x
    https://communities.sas.com/t5/Base-SAS-Programming/Finding-depth-and-last-child/m-p/428476

    https://goo.gl/7QZly9
    https://communities.sas.com/t5/SAS-Procedures/How-to-retrieve-the-last-value-in-a-quot-chain-quot-of/m-p/301260#M60593
    https://goo.gl/RkZrel
    https://communities.sas.com/t5/SAS-Procedures/How-to-retrieve-parent-child-in-a-quot-chain-quot-of/m-p/326097


    HAVE
    ===

     WORK.HAVE total obs=7          |  RULES (need to create something we can transpose)
                                    |
       FAMILY    PARENT    CHILD    |  WORK.INTERMEDIATE
                                    |
       JONES      BARB              |    FAMILY    PARENT   X    Y   (note no child column)
       JONES      BARB     PEGY     |
       JONES      PEGY     MARG     |    JONES      BARB    1    0   set y=0 and increment X. Y=0 is matriach
       JONES      MARG     BESS     |    JONES      BARB    2    0   form group x=2 and set subsequent childres
       SMITH      MARY              |    JONES      PEGY    2    1   child 1
       SMITH      MARY     JANE     |    JONES      MARG    2    2   granchild
       SMITH      JANE     JESS     |    JONES      BESS    2    3   great granchild
                                    |
                                    |    SMITH      MARY    1    0   repeat
                                    |    SMITH      MARY    2    0
                                    |    SMITH      JANE    2    1
                                    |    SMITH      JESS    2    2
                                    |
                                    |     proc transpose
                                    |      by family  x;
                                    |      id y;
                                    |      var parent;

    PROCESS (All the code)
    ======================

       data int(where=(y=0 or parent=child));
          set have;
          by family parent notsorted;
          if first.family then call missing(x, y);
          if not (first.parent and last.parent) then do; x+1; y=0;end;
          output;
          parent=Child;
          y+1;
          if parent ne '' then output;
       run;quit;

       proc transpose data=int out=obtain(drop=x _name_) prefix=C;
          by family  x;
          id y;
          var parent;
       run;quit;

    OUTPUT
    ======

      Mother JONES had a

             Daughter              BARB
             GrandChild            PEGY
             GreatGrandChild       MARG
             GreatGreatGrandChild  BESS

      Up to 40 obs WORK.WANT total obs=4

      Obs    FAMILY  Daughter  GrandChild   GreatGrandChild     GreatGreatGrandChild

       1     JONES     BARB
       2     JONES     BARB    PEGY            MARG                    BESS
       3     SMITH     MARY
       4     SMITH     MARY    JANE            JESS

    *                _               _       _
     _ __ ___   __ _| | _____     __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \   / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/  | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|   \__,_|\__,_|\__\__,_|

    ;
    data have;
      input FAMILY$ PARENT$ CHILD$;
    cards4;
     JONES BARB  .
     JONES BARB PEGY
     JONES PEGY MARG
     JONES MARG BESS
     SMITH MARY  .
     SMITH MARY JANE
     SMITH JANE JESS
    ;;;;
    run;quit;

    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __
    / __|/ _ \| | | | | __| |/ _ \| '_ \
    \__ \ (_) | | |_| | |_| | (_) | | | |
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|

    ;

    data int(where=(y=0 or parent=child));
       set have;
       by family parent notsorted;
       if first.family then call missing(x, y);
       if not (first.parent and last.parent) then do; x+1; y=0;end;
       output;
       parent=Child;
       y+1;
       if parent ne '' then output;
    run;quit;

    proc transpose data=int out=want(drop=x _name_) prefix=C;
       by family  x;
       id y;
       var parent;
    run;quit;

    *
    __      ___ __  ___
    \ \ /\ / / '_ \/ __|
     \ V  V /| |_) \__ \
      \_/\_/ | .__/|___/
             |_|
    ;

    options validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.have;
      input FAMILY$ PARENT$ CHILD$;
    cards4;
     JONES BARB  .
     JONES BARB PEGY
     JONES PEGY MARG
     JONES MARG BESS
     SMITH MARY  .
     SMITH MARY JANE
     SMITH JANE JESS
    ;;;;
    run;quit;

    %utl_submit_wps64('
    libname sd1 sas7bdat "d:/sd1";
    libname wrk sas7bdat "%sysfunc(pathname(work))";
    data int(where=(y=0 or parent=child));
       set sd1.have;
       by family parent notsorted;
       if first.family then call missing(x, y);
       if not (first.parent and last.parent) then do; x+1; y=0;end;
       output;
       parent=Child;
       y+1;
       if parent ne "" then output;
    run;quit;

    proc transpose data=int out=want(drop=x _name_) prefix=C;
       by family  x;
       id y;
       var parent;
    run;quit;
    proc print data=want;
    run;quit;
    ');
