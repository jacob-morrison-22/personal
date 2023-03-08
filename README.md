# personal
At Galen Healthcare my job was to take Clinical and Practice Management data from legacy EMR systems like Meditech Magic and Allscripts Touchworks. This data was very complex in that the schemas were incredibly messy and the size was pretty big, with some systems being over 20 TB. While a large portion of the work for an unknown system was looking at the legacy application and using SQL to locate the data in the legacy database to map to Galen’s schema, once a system was “known” the tasks necessary to archive it became fairly repetitive. The ELT process we used was run in an Azure Data Warehouse that could be fairly costly when run with unnecessary resources. With myself and my fellow coworkers in mind as customers, I created an application using Python/Flask called DORA (Database Object Recognition Assistant) because it started out as a tool to help with the initial discovery phase of the project and it was my little sister’s favorite show growing up. It eventually grew to help encompass all parts of the process through automation and . I did this all on my own, with very little input other than new features that could be added, though I mostly did that on my own as well. Some features include:
<details><summary>Full text keyword search for initial data discovery</summary>
<pre>
numDoneTimeStartDict = {}
class KeywordFindershy
	def findKeywordsByTable(self, cnxnString, sID, kw, noCols, q, thread):
	global numDoneTimeStartDict 
	while not q.empty():
		tab, columns = q.get()
		try:
			newcnxn = pyodbc.connect(cnxnString.format(SERVER, DATABASE))
		except:
			d = 'DSN={}'.format(DATABASE)
			newcnxn = pyodbc.connect(d)
		try:
			newarchcnxn = pyodbc.connect( cnxnString.format(SERVER, ARCHIEDATABASE))
		except:
			d = 'DSN={}'.format(ARCHIEDATABASE)
			newarchcnxn = pyodbc.connect(d)
		newarchcnxn.timeout = QUERYTIMEOUT
		newarchcursor = newarchenn.cursor()
		newcnxn. timeout = QUERYTIMEOUT
		newcursor = newcnxn. cursor)

		stopTable = False
		for tCol in columns:
			foundString = 'False'
			table, col, schem, dTyp = tCol

			if not stopTable:
				for KEYWORD in kw:
					if exact:
						matchKeyword = "= '{}'". format (KEYWORD)
					else:
						matchKeyword = "LIKE '%{1%'" , format (KEYWORD)
					try:
						selstring = "select count(*) from [{}].[{}].[{}] with(nolock) WHERE TRIM(LTRIM(CAST ([{}] AS VARCHAR(MAX)))) {} ".format(DATABASE, schem, table, col, matchKeyword)
						count = newcursor.execute(selstring)
						num = list(count)[0][0]


					except Exception as e:
						err = traceback.format_exc()
						print('Search Failed, {J. {). () ' format (schem, table, col))
						num = 0
						insertSearchErrors = '''INSERT INTO dora. SearchErrors (SearchID,Keyword,SchemaName,TableName,ColumnName,Error,ErrorDttm)
VALUES ('{}', '{}', '{}', '{}, '{}','{}', '{}')
'''.format(searchID, KEYWORD, schem, table, col, str(err).replace("'", "''").str(datetime.datetime.now())[:19]) 
						newarchcursor.execute(insertSearchErrors) 
						newarchcursor.commit()
						if 'query timeout expired' in str(e).lower():
							stopTable = True
					if num>0:
						foundString = 'True'
						insertSearchResults = '''
						INSERT INTO dora. SearchResults (SearchID, Keyword, SchemaName, TableName, ColumnName, FoundDttm)
						VALUES ('{}','{}','{}','{}','{}','{}')
						'''. format (searchID, KEYWORD, schem, table, col, str (datetime.datetime.now())[:19])

						newarchcursor.execute (insertSearchResults)
						newarchcursor.commit()
			numDone = numDoneTimeStartDict[SIDI(' numDone '1
			timeStart = numDoneTimeStartDict [sID][ 'timeStart' ]
			numDone += 1
			numDoneTimeStartDict[sIDI['numDone']=numDone
			timeElapsed = time.perf_counter () - timeStart
			timePerCol = timeElapsed/numDone
			colsLeft = noCols - numDone
			estimate = round ((timePerCol * colsLeft) /60, 2)
			progressBar(numDone, noCols, estimate, '', 'minutes', '() - Last Column Checked f) from Thread () - Keyword Found: () * format (searchID,
			'[{}].[{}].[{}]'.format(schem, table, col), thread, foundString))
		newcnxn.close()
		newarchcnxn.close()
		q.task_done()

</pre>
</details>

```
Collapsible Content2
```
