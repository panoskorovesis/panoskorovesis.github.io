# Google Summer of Code 2021 Paper


## Google Summer of Code 2021 Work Product Submission

### Introduction
Hello, my name is Panos Korovesis and during 2021 I was accepted in the Google Summer of Code program. I chose a [project](https://summerofcode.withgoogle.com/projects/#5587183802515456) from __LibreOffice__ and I worked under the mentorship of __Tomaž Vajngerl and Miklos Vajna__. The objective of the project was to complete the missing import/export tests of the SVM format and then separate the writing/reading functionality of the VCL Metafile to its own classes, in order to allow further refactoring of the VCL.

### Technical Overview
This project can be broken down in five distinct phases.
- __Phase One - Missing SVM Tests__: This part included the completion of the commented out methods in [svmtest.cxx](https://opengrok.libreoffice.org/xref/core/vcl/qa/cppunit/svm/svmtest.cxx?r=41ac0abb). In some cases additions were required in [mtfxmldump.cxx](https://opengrok.libreoffice.org/xref/core/vcl/source/gdi/mtfxmldump.cxx?r=8689bd54) in order for the svm file to be produced.
- __Phase Two - SvmReader__: This part included the creation of a new class [SvmReader.hxx](https://opengrok.libreoffice.org/xref/core/include/vcl/filter/SvmReader.hxx?r=a66dc788) in order to handle the reading functionality of the VCL. Reading is done with [SvmReader::Read](https://opengrok.libreoffice.org/xref/core/vcl/source/filter/svm/SvmReader.cxx?r=6189a79c#62) method, which works as follows:
    - Sets up the stream, as was previously done in [ReadGDIMetaFile](https://opengrok.libreoffice.org/xref/core/vcl/source/gdi/gdimtf.cxx?r=692c5df1#2639)
    - For each MetaAction call the created MetaActionHandler
    - The MetaActionHandler calls the appropriate read handler - which performs the reading - (e.g [PixelHandler](https://opengrok.libreoffice.org/xref/core/vcl/source/filter/svm/SvmReader.cxx?r=6189a79c#401))  for each MetaAction depending on the MetaActionType. The handlers are modified versions of the MetaAction::Read methods.
__Note__: In order for the read handlers to work, Set methods had to be created for each MetaAct’s protected members (one can be seen [here](https://opengrok.libreoffice.org/xref/core/include/vcl/metaact.hxx?r=8ede3362#361)).
- __Phase Three - SvmWriter__: This part included the creation of a new class SvmWriter.hxx in order to handle the writing functionality of the VCL. Writing is done with SvmWriter::Write method, which works as follows:
    - Sets up the stream, as was previously done in [GDIMetaFile::Write](https://opengrok.libreoffice.org/xref/core/vcl/source/gdi/gdimtf.cxx?r=692c5df1#2740)
    - For each MetaAction in the MetaFile, call the created MetaActionHandler
    - The MetaActionHandler calls the appropriate write handler - which performs the writing - (e.g [PointHandler](https://opengrok.libreoffice.org/xref/core/vcl/source/filter/svm/SvmWriter.cxx?r=7ddf7375#448)) for each MetaAction depending on the MetaActionType. The handlers are modified versions of the MetaAction::Write methods.
- __Phase Four - Replacing__: The reading/writing was performed by four methods. Those are: GDIMetaFile::Read, GDIMetaFile::Write, ReadGDIMetaFile, WriteGDIMetaFile. Whereas the new functionality is performed by SvmWriter::Write and SvmReader::Read. This part included the replacement of the method calls of the methods mentioned above with the new ones ([SvmWriter::Write](https://opengrok.libreoffice.org/xref/core/vcl/source/filter/svm/SvmWriter.cxx?r=7ddf7375#38), [SvmReader::Read](https://opengrok.libreoffice.org/xref/core/vcl/source/filter/svm/SvmReader.cxx?r=6189a79c#62))
__Note__: GDIMetaFile::GetChecksum had to be moved in SvmWriter because it was using the MetaAction::Write method.
- __Phase Five - Cleanup__: This part included the deletion of methods that were no longer in use. Those were:
    - GDIMetaFile::Write
    - GDIMetaFile::Read
    - WriteGDIMetaFile
    - ReadGDIMetaFile
    - MetaAction::Write
    - MetaAction::Read
    - GDIMetaFile::GetChecksum
 
### Commits
A list with all the commits can be found at the [bottom of this post](#commit-list)

Some __important commits__ for each phase are listed below:

__Phase One__
The commits regarding this phase start from [Add RefPoint cppunit test to vcl](https://gerrit.libreoffice.org/c/core/+/116780) and end at [Add FloatTransparent cppunit test to vcl](https://gerrit.libreoffice.org/c/core/+/117694).

Example: [Add Comment cppunit test to vcl](https://gerrit.libreoffice.org/c/core/+/117245)
- Fill the TestComment method with the required elements to be tested. They are then written in a .svm file in xml format
- Create a CheckComment method, which checks if the Comment attributes declared in the method above are the expected in the xml
- Modify mtfxmldump.cxx to format the xml output in the format we need

__Phase Two__
The commits regarding this phase start from [Create Separate SvmReader class](https://gerrit.libreoffice.org/c/core/+/118143) and end at [Add Handler for MetaAction Read](https://gerrit.libreoffice.org/c/core/+/119539).

Example: [Create Separate SvmReader class](https://gerrit.libreoffice.org/c/core/+/118143)
- Create the required files for a new class and update the appropriate makefile
- Create a constructor
- Create the Read method. It sets up the stream and calls the MetaActionHandler. At this stage the MetaActionHandler uses the original read functionality (MetaAction::Read). The next commits start replacing it with the new one.
- Update svmtest.cxx in order to use SvmReader::Read instead of GDIMetaFile::Read

Example: [Add Handler for MetaPoint Read](https://gerrit.libreoffice.org/c/core/+/118398)
- Create the PointHandler
- Create Set methods for MetaPointAction private members, as they are required by the PointHandler
- Adjust the MetaActionHandler to call the PointHandler where needed. Thus, the new read functionality is used instead of the MetaPointAction::Read

__Phase Three__
The commits regarding this phase start from [Create SvmWriter class](https://gerrit.libreoffice.org/c/core/+/119541) and end at [Add Handler for TextLineColor Write](https://gerrit.libreoffice.org/c/core/+/120372)

Example: [Create SvmWriter class](https://gerrit.libreoffice.org/c/core/+/119541)
- Create the required files for a new class and update the appropriate makefile
- Create a constructor
- Create the MetaActionHandler and the ActionHandler which replaces MetaAction::Write.
- Create the Write method. It sets up the stream and calls the MetaActionHandler. At this stage the MetaActionHandler uses the original read functionality (MetaAction::Write) for all actions except MetaAction. The next commits continue replacing with the new one.
- Update svmtest.cxx in order to use SvmWriter::Write instead of GDIMetaFile::Write

Example: [Add Handler for Ellipse Write](https://gerrit.libreoffice.org/c/core/+/119733)
- Create the EllipseHandler
- Adjust the MetaActionHandler to call the EllipseHandler where needed. Thus, the new read functionality is used instead of the MetaPointAction::Write

__Phase Four__
The commits regarding this phase start with “Replace”

Example: [Replace ReadGDIMetaFile with Svmreader::Read](https://gerrit.libreoffice.org/c/core/+/120376)
- Removes the old method call
- Creates a SvmReader object and calls SvmReader::Read, thus replacing the old Read with the new

    __Note__: These steps are repeated for each file that needs replacing

__Phase Five__
The commits regarding this phase start with "Remove". It also includes the commit [Move GDIMetaFile::GetChecksum to SvmWriter::GetChecksum](https://gerrit.libreoffice.org/c/core/+/120414)

Example: [Remove unused methods from gdimtf.hxx](https://gerrit.libreoffice.org/c/core/+/120409)
- Removes GDIMetaFile::Read
- Removes GDIMetaFile::Write
- Removes ReadGDIMetaFile
- Removes WriteGDIMetaFile

    __Note:__ This can be done, as the functionality of the removed methods is covered by SvmReader, SvmWriter.

Example: [Move GDIMetaFile::GetChecksum to SvmWriter::GetChecksum](https://gerrit.libreoffice.org/c/core/+/120414)
- Moves the method to SvmWriter while applying any changes needed in order to be compatible
- Removes the method from it's initial location

    __Note:__ This change was required as the method was using MetaAction::Write, which is replaced by the SvmWriter.

### Next Steps
The remaining steps for the completion of the project are 
- Go through the tests of the svm format trying to find edge cases and if so, write tests for them

Those steps do not need to be implemented by someone else however as I am determined to see this project to the end, even if it it goes beyond the GSoC Coding period.

### Conclusion
Working at LibreOffice was a beautiful experience. Getting in touch with people all over the world while aiming at the same goal was something new for me, but definitely something I will seek in the future! I would also like to thank the dev community for it's constant support on all my questions (silly or not) and especially my mentors Tomaž Vajngerl, Miklos Vajna for their continuous support as without them I would not be able to progress. Lastly, as I had a great experience, I plan to continue contributing to LibreOffice in the feature! Sere you all around!

### Commit List

All the commits regarding the project and thus listed below, have been __successfully merged__.

The commits are: 

1. https://gerrit.libreoffice.org/c/core/+/116780
2. https://gerrit.libreoffice.org/c/core/+/116819
3. https://gerrit.libreoffice.org/c/core/+/116967
4. https://gerrit.libreoffice.org/c/core/+/116959
5. https://gerrit.libreoffice.org/c/core/+/117153
6. https://gerrit.libreoffice.org/c/core/+/117232
7. https://gerrit.libreoffice.org/c/core/+/117245
8. https://gerrit.libreoffice.org/c/core/+/117155
9. https://gerrit.libreoffice.org/c/core/+/117368
10. https://gerrit.libreoffice.org/c/core/+/117575
11. https://gerrit.libreoffice.org/c/core/+/117702
12. https://gerrit.libreoffice.org/c/core/+/117694
13. https://gerrit.libreoffice.org/c/core/+/118143
14. https://gerrit.libreoffice.org/c/core/+/118275
15. https://gerrit.libreoffice.org/c/core/+/118217
16. https://gerrit.libreoffice.org/c/core/+/118396
17. https://gerrit.libreoffice.org/c/core/+/118398
18. https://gerrit.libreoffice.org/c/core/+/118399
19. https://gerrit.libreoffice.org/c/core/+/118403
20. https://gerrit.libreoffice.org/c/core/+/118433
21. https://gerrit.libreoffice.org/c/core/+/118473
22. https://gerrit.libreoffice.org/c/core/+/118545
23. https://gerrit.libreoffice.org/c/core/+/118550
24. https://gerrit.libreoffice.org/c/core/+/118565
25. https://gerrit.libreoffice.org/c/core/+/118566
26. https://gerrit.libreoffice.org/c/core/+/118598
27. https://gerrit.libreoffice.org/c/core/+/118608
28. https://gerrit.libreoffice.org/c/core/+/118610
29. https://gerrit.libreoffice.org/c/core/+/118633
30. https://gerrit.libreoffice.org/c/core/+/118636
31. https://gerrit.libreoffice.org/c/core/+/118637
32. https://gerrit.libreoffice.org/c/core/+/118653
33. https://gerrit.libreoffice.org/c/core/+/118665
34. https://gerrit.libreoffice.org/c/core/+/118667
35. https://gerrit.libreoffice.org/c/core/+/118672
36. https://gerrit.libreoffice.org/c/core/+/118673
37. https://gerrit.libreoffice.org/c/core/+/118674
38. https://gerrit.libreoffice.org/c/core/+/118677
39. https://gerrit.libreoffice.org/c/core/+/118698
40. https://gerrit.libreoffice.org/c/core/+/118829
41. https://gerrit.libreoffice.org/c/core/+/118833
42. https://gerrit.libreoffice.org/c/core/+/118834
43. https://gerrit.libreoffice.org/c/core/+/118836
44. https://gerrit.libreoffice.org/c/core/+/118830
45. https://gerrit.libreoffice.org/c/core/+/118837
46. https://gerrit.libreoffice.org/c/core/+/118845
47. https://gerrit.libreoffice.org/c/core/+/118892
48. https://gerrit.libreoffice.org/c/core/+/118894
49. https://gerrit.libreoffice.org/c/core/+/118895
50. https://gerrit.libreoffice.org/c/core/+/118896
51. https://gerrit.libreoffice.org/c/core/+/118955
52. https://gerrit.libreoffice.org/c/core/+/118956
53. https://gerrit.libreoffice.org/c/core/+/118957
54. https://gerrit.libreoffice.org/c/core/+/118967
55. https://gerrit.libreoffice.org/c/core/+/118968
56. https://gerrit.libreoffice.org/c/core/+/118979
57. https://gerrit.libreoffice.org/c/core/+/118980
58. https://gerrit.libreoffice.org/c/core/+/118981
59. https://gerrit.libreoffice.org/c/core/+/119092
60. https://gerrit.libreoffice.org/c/core/+/119170
61. https://gerrit.libreoffice.org/c/core/+/119171
62. https://gerrit.libreoffice.org/c/core/+/119192
63. https://gerrit.libreoffice.org/c/core/+/119193
64. https://gerrit.libreoffice.org/c/core/+/119194
65. https://gerrit.libreoffice.org/c/core/+/119537
66. https://gerrit.libreoffice.org/c/core/+/119538
67. https://gerrit.libreoffice.org/c/core/+/119539
68. https://gerrit.libreoffice.org/c/core/+/119541
69. https://gerrit.libreoffice.org/c/core/+/119580
70. https://gerrit.libreoffice.org/c/core/+/119591
71. https://gerrit.libreoffice.org/c/core/+/119592
72. https://gerrit.libreoffice.org/c/core/+/119593
73. https://gerrit.libreoffice.org/c/core/+/119594
74. https://gerrit.libreoffice.org/c/core/+/119733
75. https://gerrit.libreoffice.org/c/core/+/119734
76. https://gerrit.libreoffice.org/c/core/+/119735
77. https://gerrit.libreoffice.org/c/core/+/119736
78. https://gerrit.libreoffice.org/c/core/+/119737
79. https://gerrit.libreoffice.org/c/core/+/119738
80. https://gerrit.libreoffice.org/c/core/+/119846
81. https://gerrit.libreoffice.org/c/core/+/119898
82. https://gerrit.libreoffice.org/c/core/+/119899
83. https://gerrit.libreoffice.org/c/core/+/119900
84. https://gerrit.libreoffice.org/c/core/+/119901
85. https://gerrit.libreoffice.org/c/core/+/119902
86. https://gerrit.libreoffice.org/c/core/+/119903
87. https://gerrit.libreoffice.org/c/core/+/119920
88. https://gerrit.libreoffice.org/c/core/+/119921
89. https://gerrit.libreoffice.org/c/core/+/119922
90. https://gerrit.libreoffice.org/c/core/+/119923
91. https://gerrit.libreoffice.org/c/core/+/119924
92. https://gerrit.libreoffice.org/c/core/+/119925
93. https://gerrit.libreoffice.org/c/core/+/119963
94. https://gerrit.libreoffice.org/c/core/+/119964
95. https://gerrit.libreoffice.org/c/core/+/119965
96. https://gerrit.libreoffice.org/c/core/+/119966
97. https://gerrit.libreoffice.org/c/core/+/119967
98. https://gerrit.libreoffice.org/c/core/+/119968
99. https://gerrit.libreoffice.org/c/core/+/120106
100. https://gerrit.libreoffice.org/c/core/+/120107
101. https://gerrit.libreoffice.org/c/core/+/120108
102. https://gerrit.libreoffice.org/c/core/+/120109
103. https://gerrit.libreoffice.org/c/core/+/120110
104. https://gerrit.libreoffice.org/c/core/+/120111
105. https://gerrit.libreoffice.org/c/core/+/120153
106. https://gerrit.libreoffice.org/c/core/+/120154
107. https://gerrit.libreoffice.org/c/core/+/120155
108. https://gerrit.libreoffice.org/c/core/+/120156
109. https://gerrit.libreoffice.org/c/core/+/120157
110. https://gerrit.libreoffice.org/c/core/+/120158
111. https://gerrit.libreoffice.org/c/core/+/120256
112. https://gerrit.libreoffice.org/c/core/+/120257
113. https://gerrit.libreoffice.org/c/core/+/120258
114. https://gerrit.libreoffice.org/c/core/+/120259
115. https://gerrit.libreoffice.org/c/core/+/120263
116. https://gerrit.libreoffice.org/c/core/+/120264
117. https://gerrit.libreoffice.org/c/core/+/120306
118. https://gerrit.libreoffice.org/c/core/+/120307
119. https://gerrit.libreoffice.org/c/core/+/120308
120. https://gerrit.libreoffice.org/c/core/+/120309
121. https://gerrit.libreoffice.org/c/core/+/120312
122. https://gerrit.libreoffice.org/c/core/+/120313
123. https://gerrit.libreoffice.org/c/core/+/120349
124. https://gerrit.libreoffice.org/c/core/+/120350
125. https://gerrit.libreoffice.org/c/core/+/120351
126. https://gerrit.libreoffice.org/c/core/+/120372
127. https://gerrit.libreoffice.org/c/core/+/120376
128. https://gerrit.libreoffice.org/c/core/+/120408
129. https://gerrit.libreoffice.org/c/core/+/120409
130. https://gerrit.libreoffice.org/c/core/+/120414
131. https://gerrit.libreoffice.org/c/core/+/120471
