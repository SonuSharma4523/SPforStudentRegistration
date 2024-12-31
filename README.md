# SPforStudentRegistration

/*        
Created By: Sonu sharma       
Created Date: 26/7/2023        
Module: New Pre Examination        
Call From: Exam_Centre_Mst        
Purpose: Update Data When we want to change Exam Center Detail & Room Capacity        
*/        
       
                            
CREATE PROCEDURE ACD_StudentReg_update                                       
(              
        
@xmlDoc varchar(max),        
@Pk_studid int        
)                                                
AS     
    
--Declare @xmlDoc varchar(max)='<NewDataSet>    
--  <Exam_PreExamCenter_Mst>    
--    <Pk_PreExamId>0</Pk_PreExamId>    
--    <ExamCenterCode>ABC_898</ExamCenterCode>    
--    <ExamCenterId>1</ExamCenterId>    
--    <ExamCenterAddress>New Delhi NCR</ExamCenterAddress>    
--    <ContactDetails>9685885888</ContactDetails>    
--    <ContactMail>Amit87KJ@gmail.com</ContactMail>    
--    <ContactDetailsName>AMIT</ContactDetailsName>    
--    <FK_CityId>1436</FK_CityId>    
--    <CenterSeatCapacity>500</CenterSeatCapacity>    
--    <IsActive>true</IsActive>    
--    <FK_GeoLocationId>1299</FK_GeoLocationId>    
--    <fk_examconfigid>1</fk_examconfigid>    
--  </Exam_PreExamCenter_Mst>    
--  <Exam_CapacityRoom_Mst>    
--    <Pk_SeatCapacityCenter>0</Pk_SeatCapacityCenter>    
--    <Fk_RoomCenterId>1</Fk_RoomCenterId>    
--    <PriorityOrder>1</PriorityOrder>    
--    <CapacityAtRoom>150</CapacityAtRoom>    
--  </Exam_CapacityRoom_Mst>    
--  <Exam_CapacityRoom_Mst>    
--    <Pk_SeatCapacityCenter>1</Pk_SeatCapacityCenter>    
--    <Fk_RoomCenterId>2</Fk_RoomCenterId>    
--    <PriorityOrder>2</PriorityOrder>    
--    <CapacityAtRoom>250</CapacityAtRoom>    
--  </Exam_CapacityRoom_Mst>    
--  <Exam_CapacityRoom_Mst>    
--    <Pk_SeatCapacityCenter>2</Pk_SeatCapacityCenter>    
--    <Fk_RoomCenterId>3</Fk_RoomCenterId>    
--    <PriorityOrder>3</PriorityOrder>    
--    <CapacityAtRoom>200</CapacityAtRoom>    
--  </Exam_CapacityRoom_Mst>    
--</NewDataSet>',        
--@Pk_PreExamId int=28        
                                      
  DECLARE @ERR Int                                                                                        
  DECLARE @Idoc int                                                                                                               
                                                                                                                       
SET @ERR = 0                                                                                                  
BEGIN                                                                                             
EXEC sp_xml_preparedocument @Idoc OUTPUT, @xmlDoc                                                 
--SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;                                        
BEGIN TRY                                                                                              
BEGIN TRAN                                              
  
       
    
     
      
   
    
   Update Student_registraion_Mst Set fname=X.fname, lname=X.lname, email=X.email,          
   dept=X.dept          
   FROM OPENXML(@Idoc,'NewDataSet/Student',2)                   
   With(fname varchar(100),lname varchar(100),email varchar(100),      
   dept varchar(100))X          
   Where Student_registraion_Mst.studentid=@Pk_studid        
        
   Delete From Student_Qualification_dtl Where fk_studentid=@Pk_studid        
        
   Insert Into Student_Qualification_dtl(fk_studentid,qname,Iname,marks)              
   Select @Pk_studid,qname, iname,marks             
   FROM OPENXML(@Idoc,'NewDataSet/Student_Qualification_dtl',2)               
   With(fk_studentid int,qname varchar(100),iname varchar(100),marks varchar(100))     
       
   -- select @Pk_PreExamId    
        
      
        
   
   
  COMMIT TRAN                                            
                                                  
                                              
END TRY                                                
BEGIN CATCH                                             
IF @@TranCount > 0                                                  
ROLLBACK                                                  
DECLARE                                                  
@ErrMsg      NVARCHAR(4000),                                                  
@ErrSeverity INT;                                                  
SELECT @ErrMsg = ERROR_MESSAGE(),                                                
 @ErrSeverity = ERROR_SEVERITY()                                                
 select @ErrMsg msg, 0 success                                                 
 RAISERROR(@ErrMsg, @ErrSeverity, 1)                                                
END CATCH                                                
END 
--------------------------------
ALTER PROCEDURE ACD_StudentRegistrationInsert                                                   
(                                               
    @xmlDoc VARCHAR(MAX)                                                 
)                                                       
AS         
BEGIN                                                                                                   
    DECLARE @ERR INT, @Idoc INT, @fk_stid INT, @Result INT                                                                                                            
    SET @ERR = 0                                                                                                         

    BEGIN                                                                                                   
        -- Parse XML document  
        EXEC sp_xml_preparedocument @Idoc OUTPUT, @xmlDoc                                                         
        SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;                                              
          
        BEGIN TRY                                                                                                    
            BEGIN TRAN        

            -- Check for duplicate email
            IF EXISTS (
                SELECT 1 
                FROM Student_registraion_Mst
                WHERE email IN (
                    SELECT email FROM OPENXML(@Idoc, 'NewDataSet/Student', 2)
                    WITH (email VARCHAR(100))
                )
            )
            BEGIN
                -- Duplicate found, return without insertion
                SELECT 'Duplicate record exists' AS msg, 0 AS success;
                ROLLBACK TRAN;
                RETURN;
            END

            -- Insert data into Student_registraion_Mst  
            INSERT INTO Student_registraion_Mst(fname, lname, email, dept)             
            SELECT fname, lname, email, dept               
            FROM OPENXML(@Idoc, 'NewDataSet/Student', 2)                         
            WITH (fname VARCHAR(100), lname VARCHAR(100), email VARCHAR(100), dept VARCHAR(100));                        
             
            -- Capture newly inserted student ID  
            SET @fk_stid = SCOPE_IDENTITY();      
  
            -- Insert data into Student_Qualification_dtl  
            INSERT INTO Student_Qualification_dtl(fk_studentid, qname, Iname, marks)                    
            SELECT @fk_stid, qname, iname, marks                    
            FROM OPENXML(@Idoc, 'NewDataSet/Student_Qualification_dtl', 2)                       
            WITH (qname VARCHAR(100), iname VARCHAR(100), marks VARCHAR(100));                      

            -- Commit transaction if no errors  
            COMMIT TRAN;  
            SELECT 'Data Inserted Successfully' AS msg, 1 AS success;  
              
        END TRY  
        BEGIN CATCH                                                        
            IF @@TRANCOUNT > 0                                                        
                ROLLBACK TRAN;                                                       
  
            DECLARE                                                        
                @ErrMsg NVARCHAR(4000),                                                         
                @ErrSeverity INT;                                                          
  
            SELECT   
                @ErrMsg = ERROR_MESSAGE(),                                                       
                @ErrSeverity = ERROR_SEVERITY();                                                       
  
            -- Return error message  
            SELECT @ErrMsg AS msg, 0 AS success;                                                        
            RAISERROR(@ErrMsg, @ErrSeverity, 1);                                                       
        END CATCH;                                                       
  
        -- Clean up XML parser  
        EXEC sp_xml_removedocument @Idoc;    
    END                                                             
END




