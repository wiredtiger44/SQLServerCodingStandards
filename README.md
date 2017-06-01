# SQL Server Coding Standards
Coding standards for SQL Server development

## Table of Contents
1. [Functionality](#functionality)
2. [Error Handling](#error-handling)
3. [Naming Conventions](#naming-conventions)
4. [Code Styling](#code-styling)
5. [Commenting](#commenting)


## Functionality
<a name="Re--implementation"></a><a name="1.1"></a>
  - [1.1](#Re--implementation) **Re-implementation**: Is this functionality already implemented somewhere else? Is there something similar that can be modified?

    > Why? One set of code is easier to maintain. 

<a name="Magic"></a><a name="1.2"></a>
  - [1.2](#Magic) **No Magic Numbers and Strings**: Magic numbers or magic strings should be avoided if at all possible. A constant should be setup to represent the number or string.

    > Why? The meaning of a magic number or magic string can get lost over time. A constant or enum is clearer and ensures that if a change needs to be made it happens in only one spot 

    ```code
	/* bad  */
	SELECT	BnftPlan.BnftPlanSK
	FROM	BnftPlan
	WHERE	BnftPlan.LOBSK = 2
    
	/* good */
	DECLARE @LobSK BIGINT
	SELECT @LobSK = LobSK WHERE LOBName = 'HIX'

	SELECT	BnftPlan.BnftPlanSK
	FROM	BnftPlan
	WHERE	BnftPlan.LOBSK = @LobSK
	```
    
    <a name="InvalidParameters"></a><a name="1.3"></a>
  - [1.3](#InvalidParameters) **Invalid Parameters**: Handle invalid parameter values early in methods.

    > Why? The sooner a bad parameter is handled the less risk that the bad paramater could cause harm in the code.
    
     ```code
    /* bad - invalid parameter is not handled */
    ALTER Procedure [dbo].[spBenefitPlanSearch] @BnftPlanSK BIGINT= NULL

    AS
    BEGIN

	SELECT	
			BnftPlan.BnftPlanSK
			,BnftPlan.BnftPlanID
			,BnftPlan.BnftPlanName
	FROM	BnftPlan
	WHERE	BnftPlan.BnftPlanSK = @BnftPlanSK
    
    /* good - invalid parameter is handled */
    ALTER Procedure [dbo].[spBenefitPlanSearch] @BnftPlanSK BIGINT= NULL

    AS
    BEGIN

    IF (@BnftPlanSK IS NOT NULL)
    BEGIN
      SELECT	
          BnftPlan.BnftPlanSK
          ,BnftPlan.BnftPlanID
          ,BnftPlan.BnftPlanName
      FROM	BnftPlan
      WHERE	BnftPlan.BnftPlanSK = @BnftPlanSK
    END
    ```


<a name="UseStatements"></a><a name="1.4"></a>
  - [1.4](#useStatements) **Use Statements in functions and stored procedures**: USE and the database name should begin all stored procs and functions. In most cases the database name should not be hardcoded anywhere else in the object (Exception: if you are referring to another database)

    > Why? Makes it very specific what database the object is for. Easier for maintenance. Less confusion.

     ```code
     /* bad - no USE statement */
    ALTER Procedure [dbo].[Search] @BnftPlanSK BIGINT= NULL
    
    
    /* good - no USE statement */
    USE BenefitPlan
    ALTER Procedure [dbo].[Search] @BnftPlanSK BIGINT= NULL
    
    ```
    
## Error Handling
<a name="swallowException"></a><a name="2.1"></a>
  - [2.1](#swallowException) **Don't Swallow Exceptions**: Don't swallow exceptions without handling or logging them.

    > Why? Errors should either bubble up to a common error handling routine or be logged at the time of the error. In certain cases errors can be handled but those should be very narrowly specified.

    ```code
    /* bad (error is suppressed) */
    BEGIN TRY
       SELECT	
          BnftPlan.BnftPlanSK
          ,BnftPlan.BnftPlanID
          ,BnftPlan.BnftPlanName
      FROM	BnftPlan
      WHERE	BnftPlan.BnftPlanSK = @BnftPlanSK
    END TRY
    BEGIN CATCH;
	BREAK;
    END CATCH;
    
    	/* good - error is bubbled up to higher level */
	BEGIN TRY
		SELECT	
			BnftPlan.BnftPlanSK
			,BnftPlan.BnftPlanID
			,BnftPlan.BnftPlanName
		FROM	BnftPlan
		WHERE	BnftPlan.BnftPlanSK = @BnftPlanSK
	END TRY
	BEGIN CATCH  
		SELECT	ERROR_NUMBER() AS ErrorNumber,ERROR_MESSAGE() AS ErrorMessage; 
	END CATCH;
    ```
    
## Naming Conventions
<a name="Meaningful Names"></a><a name="3.1"></a>
  - [3.1](#meaningfulNames) **Meaningful Names**: Use meaningful Names to describe constants, variables, and methods. Don't use names that resemble keywords.

    > Why? Meaningful names help other developers quickly look at the code and understand the functionality. 

    ```code
    /* bad - name is not meaningful. resembles a keyword. */
    DECLARE  @varcharString varchar(25) = "Medical plan";
    
    /* good - name is representative of the variables representation */
    DECLARE @benefitPlanTypeMedical varchar(25) = "Medical Plan";
    ```
 <a name="Single character variables"></a><a name="3.2"></a>
  - [3.2](#singlecharacter) **No Single Character Variables**: Use meaningful Names to describe variables. Use of one character variables should be limited to loops

    > Why? Meaningful names help other developers quickly look at the code and understand the functionality. 

    ```code
    /* bad - name is not meaningful */
    DECLARE  @m varchar(25) = "Medical plan";
    
    /* good - name is representative of the variables representation */
    DECLARE @benefitPlanTypeMedical varchar(25) = "Medical Plan";

    ```

<a name="StoredProcedureNames"></a><a name="3.3"></a>
  - [3.3](#storedProcedureNames) **No Single Character Variables**: Stored Procedure names should be prefixed with "sp" (no underscore). Names should start with main table or object being referenced in rpcedure. 

    > Why? Standardizing ames helps differentiate them from other types of objects. 

    ```code
    /* bad - name does not start with sp. Doesn't reference main table */
    
     ALTER Procedure [dbo].[Search] @BnftPlanSK BIGINT= NULL
    
    /* good - name does start with sp. Does reference main table */
    ALTER Procedure [dbo].[spBenefitPlanSearch] @BnftPlanSK BIGINT= NULL
    
    /* good - name does start with spReport for a report only stored proceudre. Does reference main table */
    ALTER Procedure [dbo].[spReportBenefitPlan] @BnftPlanSK BIGINT= NULL
    ```
    
    <a name="Function names"></a><a name="3.4"></a>
  - [3.4](#functionNames) **Function Names**: Functions should be prefixed with “fn” (no underscore). They should start with a verb.  

    > Why? Standardizing method names helps differentiate them from other types of objects. 

    ```code
	/* bad - name does not start with fn and does not start with verb */
	ALTER FUNCTION [dbo].[Split] 

	/* good - name does start with fn and does start with verb */
	ALTER FUNCTION [dbo].[fnSplitString] 
    ```
    
## Code Styling
<a name="NoDebuggerStatements"></a><a name="4.1"></a>
  - [4.1](#NoDebuggerStatements) **No Debugger Statements**: Debugger statements should not be checked in. Use them temporarily and sparingly to track down problems.

    > Why? Debugger statements can cause an application to behave differently in development versus testing.

<a name="NoCommentedCode"></a><a name="4.2"></a>
  - [4.2](#NoCommentedCode) **No Commented Code**: Commented code should not be checked in. 

    > Why? Commented code can be confusing. History of code is kept in source control so commented code is not necessary. Comments that descirbe the functionality are still encouraged. 

    ```code
	/* bad - commented code isn't removed */
	DECLARE @LobSK BIGINT
	//DECLARE @LobName varchar(20)
	//DECLARE @LobSk INT
	SELECT @LobSK = LobSK WHERE LOBName = 'HIX'
	//SELECT @LobSK = LobSK WHERE LOBName = 'Commercial'
	//SELECT @LobSK = LobSK WHERE LOBSK = 2

	//SELECT *
	//FROM	BnftPlan
	//WHERE	BnftPlan.LOBSK = @LobSK
	
	SELECT	BnftPlan.BnftPlanSK
	FROM	BnftPlan
	WHERE	BnftPlan.LOBSK = @LobSK
	
	/* good - commented code is removed */
	DECLARE @LobSK BIGINT
	SELECT @LobSK = LobSK WHERE LOBName = 'HIX'
	
	SELECT	BnftPlan.BnftPlanSK
	FROM	BnftPlan
	WHERE	BnftPlan.LOBSK = @LobSK
    ```
<a name="TabsOverSpaces"></a><a name="4.3"></a>
  - [4.3](#TabsOverSpaces) **Tabs over Spaces for Indentation**: Use Tab for indentions and not Spaces.

    > Why? Tabs are more efficient... they just are. See this clip from Silicon Valley (https://m.youtube.com/watch?v=SsoOG6ZeyUI)
    
<a name="ExcessiveBlankLines"></a><a name="4.4"></a>
  - [4.4](#ExcessiveBlankLines) **Excessive Blank Lines**: Don’t use excessive blank lines to separate logical groups of code.  The suggestion is 1 blank line.

    > Why? Excessive blanks lines are inconsistent and annoying.
    
	```code
	/* bad - excessive blank lines */
	DECLARE @LobSK BIGINT
	
	
	
	
	
	SELECT @LobSK = LobSK WHERE LOBName = 'HIX'





	SELECT	BnftPlan.BnftPlanSK
	FROM	BnftPlan
	
	
	
	
	WHERE	BnftPlan.LOBSK = @LobSK
	
	/* good - no excessive blank lines */
	DECLARE @LobSK BIGINT
	SELECT @LobSK = LobSK WHERE LOBName = 'HIX'

	SELECT	BnftPlan.BnftPlanSK
	FROM	BnftPlan
	WHERE	BnftPlan.LOBSK = @LobSK
	```

<a name="RemoveUnusedCode"></a><a name="4.5"></a>
  - [4.5](#RemoveUnusedCode) **Remove Unused Code**: Code that is no longer used should be removed. Unused references should be cleaned up.

    > Why? Code that no longer is used still may need to be maintained. Less confusion if code is removed.
    
    <a name="BeginEndAtSameLevel"></a><a name="4.6"></a>
  - [4.6](#beginEndAtSameLevel) **BEGIN and END should be at same level**: BEGIN and END should be in the same level as the code outside the section.
  
	> Why? It saves a level of indenting. Complex SQL code can get unwieldy with too many levels.
    
	```code
	/* bad - BEGIN END on different level */
	IF (x=1)
		BEGIN
			SELECT Id FROM XType
		END


	/* good - BEGIN END on same level */
	IF (x=1)
	BEGIN
		SELECT Id FROM XType
	END
	```
## Commenting
<a name="CleanReadableCodeLessComments"></a><a name="5.1"></a>
  - [5.1](#CleanReadableCodeLessComments) **Clean Readable Code Less Comments**: Write clean, readable, code in such a way that it doesn’t need comments to understand.

    > Why? Comments are not a substitute for clean, readable, efficient code.
