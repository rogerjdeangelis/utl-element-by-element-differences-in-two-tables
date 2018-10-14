# utl-element-by-element-differences-in-two-tables
Element by element differences in two tables .
    Element by element differences in two tables  
    
    Additional recent enhanced solutions on end by  
                                                    
    Keintz, Mark                                    
    mkeintz@wharton.upenn.edu                       
                                                    
    data _null_,                                    
    datanull@gmail.com                              
                                                    
                                                                                                                
    Compare last years monthly grades to this years monthly grades for same students                            
                                                                                                                
      TWO SOLUTIONS                                                                                             
                                                                                                                
           1. SQL arrays                                                                                        
           2. Datastep arrays                                                                                   
           3. R       
           4. proc compare data_null                                    
           5. PDV constructions and diff function      
                                                                                                                
    github                                                                                                      
    https://tinyurl.com/y9uvbvzp                                                                                
    https://github.com/rogerjdeangelis/utl-element-by-element-differences-in-two-tables                         
                                                                                                                
    related repository                                                                                          
    https://github.com/rogerjdeangelis/utl-rename-all-columns-of-a-database-table                               
                                                                                                                
                                                                                                                
    INPUT                                                                                                       
    =====                                                                                                       
                                                                                                                
    Compute element by element differences by student                                                           
    * I have these in my autoexec;                                                                              
                                                                                                                
    %let _mon = JAN FEB MAR APR MAY JUN JUL AUG SEP OCT NOV DEC;                                                
                                                                                                                
    Not needed the solution below works with random names but PDV positions must match                          
                                                                                                                
     SD1.CURRENT total obs=5                                                                                    
                                                                                                                
      STUDENT JAN FEB MAR APR MAY JUN JUL AUG SEP OCT NOV DEC                                                   
                                                                                                                
         1    112 104 119 104 112 104 101 105 122 107 101 122                                                   
         2    104 145 148 136 120 118 116 125 114 117 112 121                                                   
         3    106 135 100 125 145 111 139 107 105 126 132 138                                                   
         4    127 148 126 121 124 147 135 117 109 143 106 115                                                   
         5    148 128 115 132 127 134 130 112 117 113 136 145                                                   
                                                                                                                
     SD1.PAST total obs=5                                                                                       
                                                                                                                
      STUDENT JAN FEB MAR APR MAY JUN JUL AUG SEP OCT NOV DEC                                                   
                                                                                                                
         1    111 137 132 143 112 112 110 121 107 131 113 132                                                   
         2    125 115 115 108 147 109 130 136 143 127 125 105                                                   
         3    144 109 138 148 116 103 112 101 138 136 146 114                                                   
         4    134 118 111 115 127 102 103 129 146 135 104 143                                                   
         5    120 143 147 131 111 108 125 113 143 147 121 115                                                   
                                                                                                                
                                                                                                                
    EXAMPLE OUTPUT                                                                                              
    --------------                                                                                              
                                                                                                                
    SD1.WANT                                                                                                    
                                                                                                                
    STUDENT JAN    FEB    MAR    APR    MAY    JUN    JUL    AUG    SEP    OCT    NOV    DEC                    
                                                                                                                
       1      1    -33    -13    -39      0     -8     -9    -16     15    -24    -12    -10                    
       2    -21     30     33     28    -27      9    -14    -11    -29    -10    -13     16                    
       3    -38     26    -38    -23     29      8     27      6    -33    -10    -14     24                    
       4     -7     30     15      6     -3     45     32    -12    -37      8      2    -28                    
       5     28    -15    -32      1     16     26      5     -1    -26    -34     15     30                    
                                                                                                                
                                                                                                                
    PROCESS                                                                                                     
    =======                                                                                                     
                                                                                                                
      1. SQL arrays                                                                                             
                                                                                                                
         %let varL=%varlist(current,drop=student);                                                              
         %array(vars,values=&varL));                                                                            
                                                                                                                
         proc sql;                                                                                              
           create                                                                                               
              table want as                                                                                     
           select                                                                                               
              %do_over(vars,phrase=%nrstr(? - P_? as ?),between=comma)                                          
           from                                                                                                 
              sd1.current as l, sd1.past(%do_over(vars,phrase=rename=(?=P_?))) as r                             
           where                                                                                                
              l.student = r.student                                                                             
         ;quit;                                                                                                 
                                                                                                                
                                                                                                                
      2. Datastep arrays                                                                                        
                                                                                                                
         %let varL=%varlist(current,drop=student);                                                              
         %array(vars,values=&varL));                                                                            
                                                                                                                
         data want;                                                                                             
           merge                                                                                                
              sd1.current                                                                                       
              sd1.past(%do_over(vars,phrase=rename=(?=P_?)));                                                   
           by student;                                                                                          
           array current &varL;                                                                                 
           array past P_:;                                                                                      
           do idx=1 to dim(current);                                                                            
              current[idx]=current[idx] - past[idx];                                                            
           end;                                                                                                 
           keep student &varL;                                                                                  
                                                                                                                
         run;quit;                                                                                              
                                                                                                                
      3. R                                                                                                      
                                                                                                                
         %utl_submit_r64('                                                                                      
         library(haven);                                                                                        
         library(SASxport);                                                                                     
         current<-read_sas("d:/sd1/current.sas7bdat")[,-1];                                                     
         past<-read_sas("d:/sd1/past.sas7bdat")[,-1];                                                           
         want_r<-current - past;                                                                                
         write.xport(want_r,file="d:/xpt/want_r.xpt");                                                          
         ');                                                                                                    
                                                                                                                
         libname xpt xport "d:/xpt/want_r.xpt";                                                                 
         data want_r;                                                                                           
            set xpt.want_r;                                                                                     
         run;quit;                                                                                              
                                                                                                                
         proc print data=want_r;                                                                                
                                                                                                                
     run;quit;                                                                                                  
                                                                                                                
    OUTPUT                                                                                                      
    ======                                                                                                      
                                                                                                                
      see above                                                                                                 
                                                                                                                
    *                _              _       _                                                                   
     _ __ ___   __ _| | _____    __| | __ _| |_ __ _                                                            
    | '_ ` _ \ / _` | |/ / _ \  / _` |/ _` | __/ _` |                                                           
    | | | | | | (_| |   <  __/ | (_| | (_| | || (_| |                                                           
    |_| |_| |_|\__,_|_|\_\___|  \__,_|\__,_|\__\__,_|                                                           
                                                                                                                
    ;                                                                                                           
                                                                                                                
    options validvarname=upcase;                                                                                
    libname sd1 "d:/sd1";                                                                                       
    data sd1.current(drop=i);                                                                                   
         retain student;                                                                                        
         array mths[12] &_mon;                                                                                  
         do student=1 to 5;                                                                                     
           do i=1 to 12;                                                                                        
              mths[i]=100 + int(50*uniform(1234));                                                              
           end;                                                                                                 
           output;                                                                                              
         end;                                                                                                   
    run;quit;                                                                                                   
                                                                                                                
    data sd1.past(drop=i);                                                                                      
         retain student;                                                                                        
         array mths[12] &_mon;                                                                                  
         do student=1 to 5;                                                                                     
           do i=1 to 12;                                                                                        
              mths[i]=100 + int(50*uniform(4321));                                                              
           end;                                                                                                 
           output;                                                                                              
         end;                                                                                                   
    run;quit;          
    
        Additional recent enhanced solutions on end by                  
                                                                    
    Keintz, Mark                                                    
    mkeintz@wharton.upenn.edu                                       
                                                                    
    data _null_,                                                    
    datanull@gmail.com                                              
                                                                    
                                                                    
    *    _       _                           _ _                    
      __| | __ _| |_ __ _        _ __  _   _| | |                   
     / _` |/ _` | __/ _` |      | '_ \| | | | | |                   
    | (_| | (_| | || (_| |      | | | | |_| | | |                   
     \__,_|\__,_|\__\__,_|  ____|_| |_|\__,_|_|_|____               
                           |_____|             |_____|              
    ;                                                               
                                                                    
    data _null_,                                                    
    datanull@gmail.com                                              
                                                                    
    proc compare base=sd1.current compare=sd1.past                  
          out=comp(drop=_:) outdiff noprint;;                       
       id student;                                                  
       run;                                                         
    proc print;                                                     
       run;                                                         
                                                                    
    *                     _                                         
     _ __ ___   __ _ _ __| | __                                     
    | '_ ` _ \ / _` | '__| |/ /                                     
    | | | | | | (_| | |  |   <                                      
    |_| |_| |_|\__,_|_|  |_|\_\                                     
                                                                    
    ;                                                               
                                                                    
    Keintz, Mark                                                    
    mkeintz@wharton.upenn.edu                                       
                                                                    
    data want;                                                      
      set sd1.past sd1.current;                                     
      by student;                                                   
      array sc {*} jan -- dec;                                      
      do _n_=1 to dim(sc);                                          
        sc{_n_}=dif(sc{_n_});                                       
      end;                                                          
      if last.student;                                              
    run;                                                            
                                                                    
                    

                                                                                                                
                                                                                                                
