# CED2AR SPSS Reader

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.1187136.svg)](https://doi.org/10.5281/zenodo.1187136)

## Artifacts

### Maven Central

[![rdb](https://maven-badges.herokuapp.com/maven-central/edu.cornell.ncrn.ced2ar.data/ced2arspssreader/badge.svg)](https://maven-badges.herokuapp.com/maven-central/edu.cornell.ncrn.ced2ar.data/ced2arspssreader) 

This project contains java classes that will allow you to read several versions of **SPSS** data sets.

It modifies the code originally written by Pascal Heus (pheus@opendatafoundation.org) for the UK Data Archive Data Exchange Tools project (http://www.data-archive.ac.uk/dext/) and the Open Data Foundation (http://www.opendatafoundation.org). Copyright 2007 University of Essex (http://www.esds.ac.uk). GNU Lesser General Public License 2.1.  Initial source: http://svn.odaf.org/ddidext/org.opendatafoundation.data/ 

This project has been worked on by different Cornell developers over the years.  The previous
developers made changes to the following files:

-  /src/org/opendatafoundation/data/spss/SPSSFile.java
-  /src/org/opendatafoundation/data/spss/SPSSNumericVariable.java
-  /src/org/opendatafoundation/data/spss/SPSSRecordType1.java
-  /src/org/opendatafoundation/data/spss/SPSSRecordType2.java
-  /src/org/opendatafoundation/data/spss/SPSSRecordType7Subtype3.java
-  /src/org/opendatafoundation/data/spss/SPSSStringVariable.java
-  /src/org/opendatafoundation/data/spss/SPSSUtils.java
-  /src/org/opendatafoundation/data/spss/SPSSVariable.java
-  /src/org/opendatafoundation/data/spss/SPSSVariableCategory.java
-  /src/org/opendatafoundation/data/xml2html.xslt

Detailed file changes are in the LGPL-Changes.txt file.


### Build

1. Clone the github repository to your machine.
2. Go to the root directory of the cloned repository.
3. Use maven 2 to build the project. On the command line, enter the following command

   ```mvn clean install -Dgpg.skip`.```  
If publishing, omit the `-Dgpg.skip`.   


### Usage 

The best way to use this code is to include the jar file in an existing project, such as [ced2arddigenerator](https://github.com/ncrncornell/ced2arddigenerator) 
The following code is in: ced2arddigenerator's SpssCsvGenerator.java file
```
	/**
	 * generates CSV using a data file uploaded by the server
	 */
	public VariableCsv generateVariablesCsv(String dataFileLocation, boolean summaryStatistics, long recordLimit) throws Exception {
		SPSSFile spssFile = null;
		File serverFile = new File(dataFileLocation);
		spssFile = new SPSSFile(serverFile);
		VariableCsv variableCSV = getVariablesCsv(spssFile, summaryStatistics, recordLimit);
		return variableCSV;
	}

	/**
	 * @return Generates two element string array. First element is a csv of
	 *         variable summary statistics Second element is a csv of variable
	 *         values as defined in SPSS Third element is the read error count
	 *         of the individual variable values. Third element is an indicator
	 *         of the validity of the statistics.
	 */
	public VariableCsv getVariablesCsv(SPSSFile spssFile, boolean includeSummaryStatistics, long recordLimit) throws SPSSFileException, IOException {
		spssFile.loadMetadata();
		List<Ced2arVariableStat> ced2arVariableStats = new ArrayList<Ced2arVariableStat>();
		int totalVariables = spssFile.getVariableCount();
		int startPosition = 0;
		for (int i = 0; i < totalVariables; i++) {
			SPSSVariable spssVariable = spssFile.getVariable(i);
			Ced2arVariableStat variable = new Ced2arVariableStat();
			variable.setName(spssVariable.getName());
			variable.setLabel(spssVariable.getLabel());
			int width = spssVariable.variableRecord.getWriteFormatWidth();
			variable.setStartPosition(startPosition);
			startPosition += width;
			variable.setEndPosition(startPosition);
			variable.setVariableNumber(spssVariable.getVariableNumber());
			if (spssVariable.categoryMap != null) {
				Iterator it = spssVariable.categoryMap.entrySet().iterator();
				while (it.hasNext()) {
					Map.Entry pair = (Map.Entry) it.next();
					SPSSVariableCategory cat = (SPSSVariableCategory) pair
							.getValue();
					if (cat.isMissing()) {
						HashMap hm = variable.getMissingValues();
						hm.put(cat.strValue, cat.label);
					} else {
						HashMap hm = variable.getValidValues();
						hm.put(cat.strValue, cat.label);
					}
				}
			}
			if (spssVariable instanceof SPSSNumericVariable) {
				variable.setNumeric(true);
				variable.setDate(spssVariable.isDate());
			} else if (spssVariable instanceof SPSSStringVariable) {
				variable.setNumeric(false);
				variable.setDate(false);
			}
			ced2arVariableStats.add(variable);
		}

		long readErrors = 0;
		if (includeSummaryStatistics) {
			readErrors = setSummaryStatistics(spssFile, ced2arVariableStats, recordLimit);
		}

		String variableStatistics = getSummaryStatisticsVaribleCSV(ced2arVariableStats, includeSummaryStatistics);
		String variableValueLabels = getVariableValueLabelCSV(ced2arVariableStats);

		VariableCsv variablesCSV = new VariableCsv();
		variablesCSV.setVariableStatistics(variableStatistics);
		variablesCSV.setVariableValueLables(variableValueLabels);
		variablesCSV.setReadErrors(readErrors);

		return variablesCSV;
	}
```
