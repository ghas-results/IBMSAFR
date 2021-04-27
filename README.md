# IBMSAFR

How to deploy to a z/OS Site

1.   Ensure the USS on that required z/OS site has GIT installed
2.   Perfrom a git clone inside USS on that site for this repo and move to that directory
3.   The Main file is the SAFRLOAD file, to allocate this as a z/OS dataset issue this USS command
      -   tsocmd "ALLOC DSN('<High Level Qualifier>.BINARY.PACKED') NEW CATALOG TRACKS UNIT(SYSDA) SPACE(900,100) LRECL(1024) RECFM(FB) DSORG(PS) BLKSIZE(27648)"   
      -   change <High Level Qualifier> to suit your site
4.   To move the SAFRLOAD file to z/OS issue this command
      -   cp  SAFRLOAD "//'<High Level Qualifier>.BINARY.PACKED'"
      -   change <High Level Qualifier> to suit your site
5.   Use TRSMAIN to unpack this new file, using this job step JCL
      
     //UNPACK   EXEC PGM=TRSMAIN,PARM=UNPACK                          
     //SYSPRINT DD SYSOUT=*                                           
     //INFILE DD DISP=SHR,DSN=<High Level Qualifier>.BINARY.PACKED        
     //OUTFILE  DD DSN=<High Level Qualifier>.BINARY.UNPACKED,UNIT=SYSDA, 
     //            DISP=(NEW,CATLG,CATLG),SPACE=(CYL,(250,25),RLSE)   
     //*   
  
6. use ADRDSSU to restore individual files in this unpacked file, using this job step JCL

     //RESTORE  EXEC PGM=ADRDSSU,REGION=4096K                      
     //SYSPRINT DD SYSOUT=*                                        
     //FILEIN   DD DISP=SHR,DSN=<High Level Qualifier>.BINARY.UNPACKED 
     //SYSIN    DD  *                                              
      RESTORE DATASET(   -                                         
        INCLUDE(SAFR.V4R18M00.BINARY.**) )    -                    
        INDDNAME(FILEIN)  -                                        
        RENAME(SAFR.V4R18M00.BINARY.**,<High Level Qualifier>.**)    
     /*      
  
