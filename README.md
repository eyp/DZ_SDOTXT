# DZ_SDOTXT
Utilities for the conversion and inspection of Oracle Spatial objects as text.
For the most up-to-date documentation see the auto-build  [dz_sdotxt_deploy.pdf](https://github.com/pauldzy/DZ_SDOTXT/blob/master/dz_sdotxt_deploy.pdf).

Generally there are few reasons for you to want to manifest Oracle Spatial objects as SQL text.  So you should only be using this code if you need to generate an example for an OTN posting or a very simple Oracle SR, or if you are exchanging a very modest amount of data with a colleague who has limited access to Oracle.  Overwhelmingly the proper way to exchange Oracle data is via datapump.  

See the [DZ_TESTDATA](https://github.com/pauldzy/DZ_TESTDATA) project as an example of what this repository can do.

##### Manifesting large Oracle Spatial geometries as sql text
In the paragraph above I state that you really should never attempt to dump anything beyond small sample geometries to text.  Unlike other databases (such as PostgreSQL) you are greatly limited as to how much information you can persist in a given block of PLSQL code.  When you submit a geometry in textual form via the object constructor, each ordinate is viewed as a node by the code parser.  So again, don't go this route.  But you are still reading!  Yes, okay sure, sometimes one may **really** want to get a large geometry into a chunk of text to share with a colleague or post as a gist or such.  But of course never do this in your production or really any workflow.  With that out of the way I provide code to allow you to crunch a geometry into a lob which can be dumped to text as hex and then afterwards reconstructed.  There are still limits but this should allow you to persist as text geometries that are much larger than what you can persist in the object constructor form.

Given a larger geometry
```sql
SELECT
dz_sdotxt_main.blob2plsql(
    dz_sdotxt_main.sdo2geomblob(a.shape)
) AS lob_text
FROM
foo a
WHERE
a.objectid = 44;
```
this will create a set of lob append statements that you can paste into a chunk of PLSQL.  The size of each hex dump line is limited to allow it flow through SQLPLUS.  Now each of these statements will utilize a couple of nodes in the code parser but far less than one node per ordinate, also the blob is compressed using UTL_COMPRESS.
```sql
DECLARE
   dz_lob BLOB
   dz_sdo MDSYS.SDO_GEOMETRY
BEGIN
   ---- START: This is the portion provided by sdo2geomblob
   DBMS_LOB.CREATETEMPORARY(dz_lob,TRUE);
   DBMS_LOB.APPEND(dz_lob,HEXTORAW('1F8B08000000000002038CBD59D2E5BAAE343622EF60DFCCE10FBF793667F0662201105A9F54F67D38716B57961645B14193489494EAFFFED76A19FFFBDFFFFDFFFC9FFF13FEE77FF97F197F9BFFF7BFFF2BE7F45F1965E7B1CFFF1D704BFFEDDD77CEE78FB5D76C903C6B4929654794857F506753441FA301517A21641CF481A474FE8342DA6EE7BFA69AAB410A7EB7F4551D525A07647683D40359EB3CCA2169CFF3AFD66E0E59F8632E36DA9E7BEF0732CA32C8EC7C8A43D24C1B2FD4FD8716DEA8E53B96D4F20464B56F48D90D33D78743F0C7369ACD4BDBBD55BCD1F01F6AC0D433A10E39937620AD657FA37A863BD6996E858CBD64FE4BFDFA44ED4C2B867B1EAF9099E4A5CF7B1824E5D9E42BEA0FF5DE30BB3377FBA13A77C21B9D3755482B98CA566B75C8DA78A352FB2FA45C485F322DF31F9086CFB88723B016CE7FB3D1D671A60AA39DC520F8D03B577FE73ACEC739FF69369B96D60AA6254F7FA1F39F862C97E510FC71367F086612BF53FD7776C13F384B512165F7BA30B7C91ED2934C525AD320E7073266DF1677EF191F118371489737DEB93984DBEE8C4D21B3E622B362D3D26BC20F4DDF6865CC3EE587A643F0D8B33C6C6E4B5D035F28DFB17479A35D1C729609F6622AFE94990119CBC6924AEAEDF19DC7C4D49D7F6B6F7456D29493C3E79F7FAE58A702C967361386379ABDD194373C074A36083E90FCD0764897A7E4EE90BAF0D2E53E450EA0F38ACD20E7FBC9E952ED8D640D9E7FB68641524DB2A37D1325F944CB766B3EFF7F956949B616D6C20BD950D76AF2BB6BDB9CAC86695B67FB1964EE219FB5FB23E48B9EB3C4DEE66CC9CEAD6A883965BFDB5179966F95DD31D63D2AF19434EB36486D5CC9C321B3C853D2BA90BC64A9AC4F481E958B707E9C60799CD7E16AD2CF33B2FC703D3BD320E7F360F19F311A6414B91F6CC19D9F398B05103B29B31CC9FB22CED72A558E413B6F4B16C89C36B77D763957CEDF18A4C8662EB539E4BC8FAC141B4A911D74D6A54DFFB9AAE46AEAD37F487650DA76831CC839AFB1981CC173E56CD08B28EDF93B7F20AD2F99DB65D37F16A8DC77F63E0D3304801D3DE763606A650E1432E5423C37A2FD4EADB297675B0EA94B302B7F43F80D8BDD1F07223F34C6BE902A2BDFCE9503C12EC421E28882F19FEBE917911D722E67FC8EED8F57885CA36E6DFCFD9DF33D320F4A4360C1AD739DDB276CBD0C5908C307DBAB4C9C2F84964AC695597D565A92A79CFF4721F51CD95CFCCB21D888EB6C3E839C3FE30B4DB37C86DC53C794C8F642153387B174FF2159A4E710B6B92D9DD6C6198C42BA980D69757B4A693C69B6EFD531E4E44F767A95B387B63C43A77F9E2D27FBD0AE667C9E8C7D981D30B8DDFD19670A64F2CD783A1059'));
   
   ...... LOTS MORE POSSIBILY .....
   
   DBMS_LOB.APPEND(dz_lob,HEXTORAW('A0D98EFD7CAE5B39C357DF06E152394B4821E740E189307552CEFEEBB2DD6D523072F986C3AEDDF3D4241FA82F87B48A655BFC7DCEED2713D0FC29E7E26B32B58EC046ADC316423AD324FBE3FC8541B60CE5BCA643120C21FC6787C80C2C9BB7B47BA219E7A3ADB210CE19AE0BE19C8A5356DC9A0E113B4EAC3685D4736A62ADD8E57D5E07C33D0F2F0639C7DB9033CEDEF918BCF24376A71E4BF598B25B4E598374F98A650C839C652AA7D3F6A74CB106B3CDEDF9E522B6462E8E90D369F868C71A6DC9AD6B2F2406CB6EC72834C8996DBC505F0E29B446CCEC4967D1D3A4F469817D7E5E28657FCAF9192CCAB3801C426BA4DA0B1D6B563EEC32B3FF4CC2A465649FA8D72257CA34136CCA2D7BBC806E633967C2C01B6C7FCA3948C460F4C96D6773E670BDCFD164556633F50EA289337406E010F16346BA90224680DB710722264DF22F748C2B590A6EDE1E08FE9CB7EDB30369134765B523F940C41BDA3E58AC16CC6DBA081E2C66459FC5C489EA76C8CD218B65DD857BB6C76AB2586CF98F213BE4D891063927B0AC16DF4462E99DCF6CA776AAF2CD70B6D9F44F4CD3B9B27D5ACE66161BED5C890E914D347C5ACE954997AA3B446E2A7C6E839CF320CB267288CCFF3166EC29E721725F63FF2A44DEA8FA3977EE399A31CB1CB33971B62C3F378E9996C45A6CF6CA537EB6F84178AED3DE87D83D36928587E2CBDA60CFDADED888CD6CCEF310D9CECD2EB373F4649AC8D54792E4F439E7AA418E4D2957EFF6755BE51A3A4BDC20B09D30FDBE871A2C9F058FC220C7B4E51EF2D347DC8B7C6C53878C2A3FEE4BAEF21ADAE63AE05D92389A7EFAD72613977DFAB164C592EBF729598CB07C7FE84CA3FCAFCD0B1C5A9C611B3F94F67FB2D86429940B918FC8F39490D1C4204917D278B61B825BE81CA7D3CF4AB1B09A9C1B802C4C9D5CE0DDBF62929DD80D32E164C94564932BCEF53986477648D9322DE94278121AA04C31D8A719AFC78FA6832E270B200346BD7C435F4F0BF6EDB195B7417ACEEB71C82D593C99A7A9408E2D21C786050BE61AE2E620D4639024475878CA1FC8B119E487BE007D8B570643FE1B32E913FB1DF317B2C6A4BDFE0D81F72A56CD1DACACE2E973CB000A36A23F65263A5DCB21459E927CFAFF42CE2FC805E2FB6389C70BA04364A99CBF299F90B39A64CF587CE440B06C53F7E57456A0F80E79DCB14CB92D66BF902A5F68D66F48E653FAFE1A4B9B4B56FBB8F3D2C59F6DA93924CBA5BEB2FF50930F901D90F811EFC4FD00CE91B3A2573617E327DD37C7B14F647B6FB724569753708DEE9063DBC957F65D28374EBEDFA79DA995A1DC852D66740E2F7CAC1A1EDA77516230ED4E5B958057F044CF269B1205D836DC8A5912C3C79F2226639A735F88DCF0D32D'));

   ---- DONE
   
   dz_sdo := dz_sdotxt_main.geomblob2sdo(dz_lob);
   
   -- Now do something with the geometry
   
END;
/
```
Again I am not entirely sure of the upper limit of this technique in terms of the number of ordinates.  Drop me a line if you sort it out.  The above solution uses the secret MDSYS.SDO_UTIL.TO_CLOB and MDSYS.SDO_UTIL.FROM_CLOB functions to dump and reconstruct the geometry object.  This **should** work for all forms and types of geometries as it just a manifestation of the VARRAYs as bar-delimited text.  But the recipiant of the sql text dump will either need geomblob2sdo or the logic of that function to reconstruct the geometry.  Take a look at the function, it's not that complicated and could be easily put into the declaration of your PLSQL block.  However it is a small complication.

Another approach is if your geometry is compliant with the OGC Simple Features specification 1.0 - e.g. simple 2D straightline geometries, you can use WKB as the blob format to pass around.  Note that WKB is not compressed as far as I can tell so it may be larger than the sdo2geomblob solution.  But the reconstruction step is much simpler.  
```sql
SELECT
dz_sdotxt_main.blob2plsql(
    MDSYS.SDO_UTIL.TO_WKBGEOMETRY(a.shape)
) AS lob_text
FROM
foo a
WHERE
a.objectid = 44;
```
Then to reconstruct things just pass the blob into the geometry constructor
```
dz_sdo := MDSYS.SDO_GEOMETRY(dz_lob,4269);
```
Just make sure to pass along the correct SRID to the constructor as WKB does not persist that information.  Again this approach will work for the usual "run of the mill" geometries but is not as robust as the sdo2geomblob solution.
   
## Installation
Simply execute the deployment script into the schema of your choice.  Then execute the code using either the same or a different schema.  All procedures and functions are publically executable and utilize AUTHID CURRENT_USER for permissions handling.

## Collaboration
Forks and pulls are **most** welcome.  The deployment script and deployment documentation files in the repository root are generated by my [build system](https://github.com/pauldzy/Speculative_PLSQL_CI) which obviously you do not have.  You can just ignore those files and when I merge your pull my system will autogenerate updated files for GitHub.

## Oracle Licensing Disclaimer
Oracle places the burden of matching functionality usage with server licensing entirely upon the user.  In the realm of Oracle Spatial, some features are "[spatial](http://download.oracle.com/otndocs/products/spatial/pdf/12c/oraspatitalandgraph_12_fo.pdf)" (and thus a separate purchased "option" beyond enterprise) and some are "[locator](http://download.oracle.com/otndocs/products/spatial/pdf/12c/oraspatialfeatures_12c_fo_locator.pdf)" (bundled with standard and enterprise).  This differentiation is ever changing.  Thus the definition for 11g is not exactly the same as the definition for 12c.  If you are seeking to utilize my code **without** a full Spatial option license, I do provide a good faith estimate of the licensing required and when coding I am conscious of keeping repository functionality to the simplest licensing level when possible.  However - as all such things go - the final burden of determining if functionality in a given repository matches your server licensing is entirely placed upon the user.  You should **always** fully inspect the code and its usage of Oracle functionality in light of your licensing.  Any reliance you place on my estimation is therefore strictly at your own risk.

In my estimation functionality in the DZ_SDOTXT repository requires the full Oracle Spatial option for 10g, 11g and 12c.
