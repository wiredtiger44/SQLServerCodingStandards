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
  - [3.3](#StoredProcedureNames) **Stored Procedure Names**: Stored Procedure names should be prefixed with "sp". Don't start procedures with "sp_".

    > Why? Standardizing class names helps differentiate them from other types of objects. 

    ```code
    /* bad - name is not Pascal case, is a verb and does not match file name. */
    In file exporting.cs
    public class exportingdownloading
    
    /* good - name is Pascal case, is a noun and matches file name. */
    In file BenefitPlan.cs
    public class BenefitPlan
    ```
    
    <a name="Method names"></a><a name="3.4"></a>
  - [3.4](#methodNames) **Method Names**: Use Pascal casing for Method Names.  Method names should start with a verb and tell what it does. 

    > Why? Standardizing method names helps differentiate them from other types of objects. 

    ```code
    /* bad - name is not Pascal case and does not start with verb */
    public string benefitPlan(long benefitPlanSK)
    {
    }
    
    /* good - name is Pascal case and start with verb */
    public string GetBenefitPlan(long benefitPlanSK)
    {
    }
    ```
    
      <a name="Variable and Parameter names"></a><a name="3.5"></a>
  - [3.5](#VariableParameterNames) **Variable and Parameter Names**: Use Lower Camel casing for variables and method parameters.  

    > Why? Standardizing method names helps differentiate them from other types of objects. 

    ```code
    /* bad - name is not Camel case */
    public string benefitPlan(long benefitplansk)
    {
      string benefitplantype;
    }
    
    /* good - name is Camel case */
    public string GetBenefitPlan(long benefitPlanSK)
    {
      string benefitPlanType;
    }
    ```
    
      <a name="Interface, View Model, repository, BLL, DAL, Controller names"></a><a name="3.6"></a>
  - [3.6](#interfaceNames) **Interface, View Model, repository, BLL, DAL, Controller Names**: Use Pascal casing and begin with a prefix for the following type of class: Interface(I). Use Pascal casing and end with a Suffix for the following types of classes: ViewModel ("VM"), Repository ("Repository"), BLL ("BLL"), and Controller ("Controller")

    > Why? Standardizing method names helps differentiate them from other types of objects. 

    ```code
    /* bad - name is not Pascal case, does not start with "I", and is not a noun */
    public interface processing
    {
    }
    
    /* bad - name is not Pascal case, does not end with "BLL" */
    public class businesslayerBenefitPlan
    {
    }
    
    /* good - name is Pascal case , starts with an "I", and is a noun */
    public interface IBenefitPlan
    {
    }
    
    /* good - name is Pascal case, ends with "BLL" */
    public class BenefitPlanBLL
    {
    }
    ```
    
    <a name="Boolean/Bit variables"></a><a name="3.7"></a>
  - [3.7](#VariableParameterNames) **Boolean/Bit Variable Names**: Use Lower Camel casing for variables and method parameters. Prefix with "is" or similar.

    > Why? Standardizing method names helps differentiate them from other types of objects. The "is" prefix makes it clear what value is true and what value is false.

    ```code
    /* bad - name is not lower Camel case and is not prefixed with "is" or similar */
    bool family;
    
    
    /* good - name is lower Camel case and is prefixed with "is" or similar */
    bool isMemberOfFamily;
    ```
    
## Code Styling
<a name="NoDebuggerStatements"></a><a name="4.1"></a>
  - [4.1](#NoDebuggerStatements) **No Debugger Statements**: Debugger statements should not be checked in. Use them temporarily and sparingly to track down problems.

    > Why? Debugger statements can cause an application to behave differently in development versus testing.

<a name="NoCommentedCode"></a><a name="4.2"></a>
  - [4.2](#NoCommentedCode) **No Commented Code**: Commented code should not be checked in. 

    > Why? Commented code can be confusing. History of code is kept in source control so commented code is not necessary. Comments that descirbe the functionality are still encouraged. 

    ```code
    /* bad - a bunch of commented code */
    //string mmm = "Medical Plan";
    //int mmm = 1;
    
    string benefitPlanTypeMedical = "Medical Plan";
    //if (mmm == "Medical plan");
    //if (mmm == 1);
    if (bnftPlanType == benefitPlanTypeMedical)
    {
    }
    
    /* good - name is representative of the variables representation */
    // check to see if the benefit plan type is Medical Plan
    string benefitPlanTypeMedical = "Medical Plan";
    if (bnftPlanType == benefitPlanTypeMedical)
    {
    }
    ```
<a name="TabsOverSpaces"></a><a name="4.3"></a>
  - [4.3](#TabsOverSpaces) **Tabs over Spaces for Indentation**: Use Tab for indentions and not Spaces.

    > Why? Tabs are more efficient... they just are. See this clip from Silicon Valley (https://m.youtube.com/watch?v=SsoOG6ZeyUI)
    
<a name="ExcessiveBlankLines"></a><a name="4.4"></a>
  - [4.4](#ExcessiveBlankLines) **Excessive Blank Lines**: Don’t use excessive blank lines to separate logical groups of code.  The suggestion is 1 blank line.

    > Why? Excessive blanks lines are inconsistent and annoying.
    
    ```code
    /* bad - several blank lines for separation */
    string benefitPlanType = "Medical";
    string benefitPlanType2 = "Medical2";
    
    
    
    
    
    
    return benefitPlantype + benefitPlanType2;
    
    /* good - only one blank line for separation */
    string benefitPlanType = "Medical";
    string benefitPlanType2 = "Medical2";

    return benefitPlantype + benefitPlanType2;
    ```

<a name="UseVisualStudioFormatDocument"></a><a name="4.5"></a>
  - [4.5](#UseVisualStudioFormatDocument) **Use Visual Studio Format Document**: Use Visual Studio Format Document functionality (Ctrl + K, Ctrl + D) to format files. 

    > Why? Ensures consistent formatting of the code levels. 

a name="RemoveUnusedCode"></a><a name="4.6"></a>
  - [4.6](#RemoveUnusedCode) **Remove Unused Code**: Code that is no longer used should be removed. Unused references should be cleaned up.

    > Why? Code that no longer is used still may need to be maintained. Less confusion if code is removed.
    
## Commenting
<a name="CleanReadableCodeLessComments"></a><a name="5.1"></a>
  - [5.1](#CleanReadableCodeLessComments) **Clean Readable Code Less Comments**: Write clean, readable, code in such a way that it doesn’t need comments to understand.

    > Why? Comments are not a substitute for clean, readable, efficient code.


<a name="XMLDocumentation"></a><a name="5.2"></a>
  - [5.2](#XMLDocumentation) **XML Documentation for API End Points**: For controllers, add XML documentation to each API endpoint with “///”. Describe what the API endpoint expects and what it returns.

    > Why? Consumers of the API will have documentation about what and how an API works.
    
    ```code
    /* bad - no XML Documentation */
    [HttpGet]
    public IHttpActionResult Search(string bnftName = null, string bnftCode = null, long? svcTypeSK = null)
    {
    }
    
    /* good - XML documentation documenting method */
    /// <summary>
    /// Benefit Definition Search
    /// </summary>
    /// <param name="bnftName">the bnft name</param>
    /// <param name="bnftCode">the bnft Code</param>
    /// <param name="svcTypeSK">the Service Type SK</param>
    /// <returns>list of Benefits with their optional Service Types</returns>
    [HttpGet]
    public IHttpActionResult Search(string bnftName = null, string bnftCode = null, long? svcTypeSK = null)
    {
    }
    ```
