# Dora Explainer
At Galen Healthcare my job was to take Clinical and Practice Management data from legacy EMR systems like Meditech Magic and Allscripts Touchworks. This data was very complex in that the schemas were incredibly messy and the size was pretty big, with some systems being over 20 TB. While a large portion of the work for an unknown system was looking at the legacy application and using SQL to locate the data in the legacy database to map to Galen’s schema, once a system was “known” the tasks necessary to archive it became fairly repetitive. The ELT process we used was run in an Azure Data Warehouse that could be fairly costly when run with unnecessary resources. With myself and my fellow coworkers in mind as customers, I created an application using Python/Flask called DORA (Database Object Recognition Assistant) because it started out as a tool to help with the initial discovery phase of the project and it was my little sister’s favorite show growing up. It eventually grew to help encompass all parts of the process through automation and . I did this all on my own, with very little input other than new features that could be added, though I mostly did that on my own as well. Some features include:

<details><summary>Full text keyword search for initial data discovery - Example included</summary>
<pre>
numDoneTimeStartDict = {}
class KeywordFinder:
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

<details><summary>Mass transformation of sql scripts to be adjusted for new clients and for features that were in local SQL Server but not Azure Data Warehouse
</summary>
<pre>
</pre>
</details>

<details><summary>Managing Azure resources so they were running at the proper scale and would turn off if idle - Example included
</summary>
<pre>
class AzureManager:
	def __init__(self,customer,external,clientid='',keyvault=''):
		#self.sqllitecnxn = createConnection('azureman.db')
		co.pullConfiguration(customer,external,clientid,keyvault)
		self.IdleTimerRunning = co.IdleTimerRunning
		self.latest = co.latest

	def run(self,cmd):
		completed = subprocess.run(["powershell", "-Command",cmd], caputre_output=True)
		return {'Error':co.convertHtmlString(completed.stderr.decode('utf-8')), 'Output':co.convertHtmlString(completed.stdout.decode('utf-8')),'Command':completed.args[2:]}

	def runAzureCommand(self,cmd='',paramObj={},scale='DW100c',updIdleStat=True):


		idletime = 1
		if paramObj:
			cmd = paramObj.get('command','')
			scale = paramObj.get('scale','')
			idletime = paramObj.get('idle',1)

		self.error = None
		dbInfo = co.dbs.get(co.resourceGroup,co.server,co.database)
		updIdleStatString = ''
		if cmd.lower() == 'update':
			ret = {'Output':'Idle Time Check','Error':'','Command':'Update'}
		if updIdleStat:
			updIdleStatString = ",IdleTimerStatus = ''"
		if cmd.lower() in ['resume', 'pause']:
			if cmd == 'Pause' and dbInfo.status == 'Online':
				func = co.dbs.begin_pause
			elif cmd == 'Resume' and dbInfo.status == 'paused':
				func = co.dbs.begin_resume
			else:
				return{'Output':'','Error':'Cannot {} database while it is {}'.format(cmd.lower(),dbInfo.status),'Command':''}
			self.currentAction = cmd[:-1]+'ing'

			print(self.currentAction)
			try:
				self.poll = func(co.resourceGroup,co.server,co.database)
			except Exception as e:
				self.error = str(e)

			#print('Pausing')
			dbInfo = co.dbs.get(co.resourceGroup,co.server,co.database)
			if updIdleStat:
				co.runStringQuery(azdb,"UPDATE AzureConfiguration SET UpdateDttm = '{}', Status = '{}' {} {}".format(datetime.datetime.now(),dbInfo.status,updIdleStatString,co.cusExtWhere))
			while not self.poll.done():
				dbInfo = co.dbs.get(co.resourceGroup,co.server,co.database)
				# print(poll.status())
				co.runStringQuery(azdb,"UPDATE AzureConfiguration SET Status = '{}' {}".format(dbInfo.status,co.cusExtWhere))
				time.sleep(1)
			co.runStringQuery(azdb,"UPDATE AzureConfiguration SET Status = '{}' {}",format(dbInfo.status,co.cusExtWhere))

			output = self.poll.status()
			ret = {'Output':str(output),'Error':self.error,'Command':str(func)}
		elif cmd.lower() == 'scale':
			if dbInfo.status != 'Online':
				return {'Output':'', 'Error':'Cannot {} database while it is {}'.format(cmd.lower(),dbInfo.status),'Command':''}


			# if cmd == 'ScaleUp':
			#	scale = 'DW300c'
			# elif cmd == 'ScaleDown':
			# scale = 'Dw100c'

			runCmd = 'Add-AzAccount -identity\nSet-AzSqlDatatbase -ResourceGroupName "{r}" = DatabaseName "{d}" -ServerName "{s}" -RequestedServiceObjectiveName "{sc}"'.format(r=co.resourceGroup,d=co.database,s=co.server,sc,scale)
			#print(runCmd)
			ret =self.run(runCmd)
			time.sleep(5)
			dbInfo = co.dbs.get(co.resourceGroup,co.server,co.database)

		if cmd.lower() in ['scale','resume','update'] and dbInfo.status == 'Online':
			if not self.IdleTimerRunning:
				print('Starting Idle Timer')
				self.startIdleTimeoutFinal(idletime)

			# updateIdleStatString = ",IdleTimerStatus = 'Running'"
		# if updIdleStat:
		co.runStringQuery(azdb,"UPDATE AzureConfiguration SET UpdateDttm = '{}' {} {}".format(datetime.datetime.now(),dbInfo.status,updIdleStatString,co.cusExtWhere))

			# print(ret)

		print('Finished')
		return ret 
	#def getSeriveObjectiveInfo(self):


	def startIdleTimeoutFinal(self,idleHours=1,paramObj={}):
		IGNOREQUERIES = ['select @@spid;','use[{}]'.format(co.database),'begin tran', 'rollback','select getdate() as dt', 'select *\nfrom [etl_dbo].[job]']
		archieQueries = ["List of queries to ignore because they are run by Archie"]
		if not idleHours:
			idleHours = 1
		if paramObj:
			# cmd = paramObj.get('command','')
			idleHours = paramObj.get('idle','1')
			#SERVERPAUSED = False
			noQueries = 0
			#am.runAzureCommand('Connect')
			r = None
			IDLETIMEOUT = int(idleHours) * 60
			RESTMINUTES = 10
			INTERVALS = IDLETIMEOUT/RESTMINUTES
			RESTSECONDS = RESTMINUTES * 60
			WHOLEINTERVALS = math.floor(INTERVALS)
			LASTINTERVAL = RESTSECONDS * (INTERVALS%1)
			# RESTSECONDS = RESTMINUTES * 60
			
			# last = time.perf_counter()
			# prevRes = [{'':1}]
			# FIRST = TRUE

			intervalsLeft = WHOLEINTERVALS
			lastInterval = LASTINTERVAL
			restSeconds = RESTSECONDS
			if self.IdleTimerRunning:
				print('Idle Timer already running')
				start = False
			else:
				try:
					print('Testing Server Connection:', end = ' ')
					# print(co.ARCHIECNXNSTRING)
					pyodbc.connect(co.ARCHIECNXNSTRING)
					err = ''
					start = True

					print('Success')
					startDate = datetime.datetime.now()
					idleStatus = 'Starting Idle Timer\n Timer Started: {}\nTimer Updated: {}'.format(startDate,startDate)
					co.runStringQuery(azdb,"UPDATE AzureConfiguration SET UpdateDttm = '{}' IdleTimerStartDttm = '{}', IdleTimerUpdateDtt='{}' , IdleTimerStatus = '{}' {}".format(startDate,startDate,startDate,idleStatus,co.cusExtWhere))

					SERVERPAUSED = False
					lastPollTime = co.buildTableDict(co.ARCHIECNXNSTRING,runStr='select getDate() as Dt')[0]['DT']
				except Exception as e:
					start = False
					err = str(e)
					print(e)

			if start:
				lastPollTime = co.buildTableDict(co.ARCHIECNXNSTRING,runStr='select getDate as Dt')[0]['Dt']
				idleStatus = 'Idle Timer running, server will pause in {} minute if no activity is detected.'.format(IDLETIMEOUT)
				print(idleStatus)
				co.runStringQuery(azdb,"UPDATE AzureConfiguration SET IdleTimerStartDttm='{}', IdleTimerStatus= '{}' {}".format(datetime.datetime.now(),idleStatus,co.cusExtWhere))

				time.sleep(restSeconds)
				self.IdleTimerRunning =True
				while True:
					r = None
					dbInfo = co.dbs.get(co.resourceGroup,co.server,co.database)
					ltst = co.latest
					if dbInfo.status == 'Online':
						try:
							# now = time.perf_counter()

							# if now = last > 15 or FIRST:
							idleStatus = 'Checking Server Activity\nTimer Started: {}\nTimer Updated: {}'.format(startDate,datetime.datetime.now())
							co.runStringQuery(azdb,"UPDATE AzureConfiguration SET IdleTimerStartDttm='{}', IdleTimerStatus = '{}' {}".format(datetime.datetime.now(),idleStatus.co.cusExtWhere))

							# print('Checking Server Activity:',end = ' ')
							res = co.buildTableDict(co.ARCHIECNXNSTRING,runStr=''' SELECT start_time,end_time,command,status
				From sys.dm_pdw_exec_requests (NOLOCK) -- azuremanageridlequerytag
				WHERE 1=1 -- and status not in ('Completed', 'Failed', 'Cancelled')
					AND session_id <> session_id()
					AND (start_time > '{}' or end_time is null) order by start_time desc ''' .format(str(lastPollTime)[:19]))
							useRes = [row for row in res if row['command'].lower().strip() not in IGNOREQUERIES and '--azuremanageridlequerytag' not in row['command'].lower().strip()]
							# last = time.perf_counter()
							# FIRST = False

							lastPollTime += datetime.timedelta(seconds=restSeconds)
							# else:
							# res = prevRes
						except Exception as e

							# res = [{'c':0}]
							useRes = []
							if 'Cannot connect to database when it is paused' in str(e):
								SERVERPAUSED = True
							else:
								idleStatus = 'Error when connecting to DB\n'
								idleStatus += str(e)
								#print(e)
								co.runStringQuery(azdb,"UPDATE AzureConfiguration SET IdleTimerUpdateDttm='{}', IdleTimerStatus='{}' {}".format(datetime.datetime.now(),idleStatus,co.cusExtWhere))
								
								print(idleStatus)
								print(co.ARCHIECNXNSTRING)

						else:

							if len(useRes) > 0:
								idleStatus = 'Activity detected\nTimer Started: {}\nTimer Updated: {}'.format(startDate,datetime.datetime.now())
								co.runStringQuery(azdb,"UPDATE AzureConfiguration SET IdleTimerUpdateDttm='{}', IdleTimerStatus='{}' {}".format(datetime.datetime.now(),idleStatus,co.cusExtWhere))

								noQueries = 0
								intervalsLeft = WHOLEINTERVALS
								lastInterval = LASTINTERVAL
								restSeconds = RESTSECONDS
								# y=input('stop? ')
								# if y.lower() in ['y','yes']:
				#					break
							else

								if intervalsLeft:
									noQueries += 1
									intervalsLeft -= 1
							
							# if noQueries > 0:
								# intervalsLeft = INTERVALS - noQueries


								noAct = round((noQueries * RESTSECONDS)/60,1)
								if restSeconds != RESTSECONDS:
									noAct += round((restSeconds)/60,1)
								leftAct = round((intervalsLeft * RESTSECONDS)/60 + (lastInterval/60),1)
								idleStatus = 'No activity detected for the last {} minutes. Server will pause in {} minutes \n Timer Started:{}\nTimer Updated: {}'.format(noAct,leftAct,startDate,datetime.datetime.now())
								co.runStringQuery(azdb,"UPDATE AzureConfiguration SET IdleTimerUpdateDttm='{}', IdleTimerStatus='{}' {}".format(datetime.datetime.now(),idleStatus,co.cusExtWhere))
							
							# else:
							
							#print(intervalsLeft)
							#print(lastInterval)
							#print(restSeconds)
							#print(noQueries)
							#print(SERVERPAUSED)
							if not intervalsLeft and not SERVERPAUSED:
								if not lastInterval:
									idleStatus = 'Idle Timer has ended, scaling down and pausing server\nTimer Started: {}\nTimer Updated: {}'.format(startDate,datetime.datetime.now())
									co.runStringQuery(azdb,"UPDATE AzureConfiguration SET IdleTimerUpdateDttm='{}', IdleTimerStatus='{}' {}".format(datetime.datetime.now(),idleStatus,co.cusExtWhere))
									
									print('Scaling Down Server')
									r =  self.runAzureCommand('Scale', updIdleStat=False)
									print('Pausing Server')
									r = self.runAzureCommand('Pause', updIdleStat=False)
									print('Server Paused')
									break
								else:
									restSeconds = lastInterval
									lastInterval = 0
						# print(noQueries)
						# prevRes = res
						time.sleep(restSeconds)
				idleStatus = 'Server has been scaled down and paused\n Timer Updated: {}'.format(startDate,datetime.datetime.now())
				co.runStringQuery(azdb,"UPDATE AzureConfiguration SET IdleTimerUpdateDttm='{}', IdleTimerStatus='{}' {}".format(datetime.datetime.now(),idleStatus,co.cusExtWhere))
				self.IdleTimerRunning = False
</pre>
</details>

<details><summary>Plugins to wrangle messy data that couldnt be translated with only SQL scripts</summary>
<pre>
</pre>
</details>

<details><summary>Orchestrate running of extraction sql scripts</summary>
<pre>
</pre>
</details>

<details><summary>Updating configuration settings for orchestration of load scripts  - Example included</summary>
<pre>
def getFormDictsByTask(taskDict,jobID,defDict={}):

	timeZones = ['Atlantic Standard Time', 'Eastern Standard Time', 'Central Standard Time', 'Mountain Standard Time', 'Pacific Standard Time', 'Alaskan Standard Time', 'Hawaiian Standard Time']
	itemTypes = getItemTypes ()
	trueFalseOpts = ['True', 'False']
	trueFalseBlankOpts = [''] + trueFalseOpts
	singleChoiceInputs = {'desiredTimeZone' :timeZones, 'skipValidation' :trueFalseOpts, 'environment' : ['Staging', 'Production '],
						 'includeComments':trueFalseBlankOpts, 'includeTypes' :trueFalseBlankOpts, 'includeDiscrete' :trueFalseBlankOpts}
	numericInputs = {'batchSize' :{'max' :1000, 'min' :1}, 'concurrentQueries': {'max' :16, 'min' :1}}
	multChoiceInputs = {'includeItemTypes' :itemTypes, 'excludeItemTypes': itemTypes}
	allInputs = {}
	for inp in singleChoiceInputs:
		ch = singleChoiceInputs[inp]
		allInputs[inp] = {'typ': 'select', 'choices':ch}
	for inp in multChoiceInputs:
		ch = multChoiceInputs[inp]
		allInputs[inp] = {'typ': 'multiSelect', 'choices':ch}
	for inp in numericInputs:
		ch = numericInputs[inp]
		allInputs[inp] = {'typ': 'number', 'attributes': ch}
	order = 1
	formDictsByTask = {}
	for taskKey, taskFields in taskDict.items():
	
		itindict = {}
		formDict = {}
		itinDict = getstuff(taskKey, 'dict')
		for key, value in itinDict.items():
		
			default = ''
			typeChoicesDict = allInputs.get (value, ('typ': 'string', 'size': '40'))

			for i in taskFields:
				if str(i.name).lower() == value.lower():
					default = defDict.get (value,'')

					if i.string != None and not default:
						default = str(i.string)
					if value. lower in ['excludetypes', 'includetypes ']:
						default = default.split('|')
	
		valDict = {'name' :value, 'label' :value, 'current' :default, 'order' :order, 'number': jobID}
	
		valdict.update(typeChoicesDict)
		formDict[value] = valdict
		order += 1
	
		formDictsByTask[taskKey]= formDict 
	return formDictsByTask

@app.route("/createitinerary", methods=[ 'POST', 'GET'])
def createitinerary():
	#goto:itinerary
	environment = request.form.get('environment', 'staging')
	envFull = 'Staging'
	if environment == 'prod':
		envFull = 'Production'
	choice = request.form.get('selectType')
	itinplugins = request.form.getlist('plugins'‚ None)
	extractplugins = []
	if choice == 'discrete':
		plugins = ['List Of Discrete Plugin UUIDs']
		if co.SOLTYPE == 'Warehouse'
			itinplugins = ['UUIDs']
			extractplugins = ['UUIDs']
	elif choice == 'nondiscrete':
		itinplugins=['List Of Nondiscrete Plugin UUIDs']
		if co.SOLTYPE == 'Warehouse':
			itinplugins = ['UUIDs' ]
			extractplugins = ['UUIDs']
	elif choice == 'person':
		itinplugins = ['UUIDs']
	elif choice == 'pavor';
		itinplugins = ['UUIDs']
	elif choice == 'nonpatientmessage':
		itinplugins = ['UUIDs']

	elif choice == 'nonpatientaudit':
		itinplugins = ['UUIDs']
	pluginsByType = {'Itinerary' :itinplugins}
	session['t'] = choice
	session['plugins'] = plugins
	session['environ'] = environment
	if plugins:
		try:
		
			taskDict = {}
			itinDictByTask = {}
			for plugin in plugins:
				plugin = plugin.upper()
				pid = plugin. lower()
				plugName = PLUGINDICT.get(pid, [])
				if plugName:
					plugKey = plugName[0] + '<br›‹br›' + plugin + '‹br›‹br>' + plugName [2]
				#print (plugin)
				itinDict = getStuff(plugin, 'dict')
				xml = getstuff (plugin, 'xml') 
				try:
					children = xml.task.find_all()
				except:
					children = xml.extractor.find_all()
				fields = []
				for child in children:
					fields.append(child)
				fields = list(filter (lambda x: × != '\n' ,fields))
				taskDict[pid] = fields
				itinDictByTask[Markup(plugKey)] = itinDict
				optionalFields = {}
				for tag in children:
					#The following statement fixes an issue where the plugin example has unknown in the excludeItemTypes field
					#If dev updates the plugin example and removes the unknown the following two Lines can be removed 
					if tag.name == 'excludeItemTypes' and tag,string == 'Unknown':
						optionalFields[tag.name] = ''
					elif tag.string != None:
						optionalFields [tag.name] = tag.string
				defaultDict = {"DICTIONARY OF DEFAULT VALUES FOR":"Archie Configuration"}
				for field in ['cAuthEndpointUri', 'vcoApiEndpointuri', 'environment ']:
					if field in optionalfields:
						defaultDict[field] = optionalFields[field]
			formDictsByTask = getformDictsByTask(taskDict,'', defaultdict)
			formDictByXMLType= {'Itinerary': formDictsByTask} 
			formDictByJob ={'new': formDictByXMLType} 
			previewXML = buildonclickButton('preview_()_(' .format ('Itinerary', 'new'), 'updateXMLFromJob(event)', 'Preview Itinerary')
			formForNewJob = previewXML+ '<br› '+buildFormForJob(itnExtDict=formDictByJob).get('new', {}).get('Itinerary','')
			# elif 1==2:
		except Exception as e:
			form, formi, jobTable = displayitinerary()
			err= 'Error getting plugins (make sure to run Archie with the correct ArchivalETL DB in the config file at least once): (1'.format(e)
			return render template('DORAsetitinerary.htmi',title= 'DORA', form-form, form1=form1, result-jobTable, error=err)
		else:
			form1 = itineraryForm()
			return render_template('DORAcreateitinerary.htmI',title = 'DORA', result=Markup(formForNewJob),navBar=Markup (NAVBAR))


</pre>
</details>

<details><summary>Configuring the front end of the cloud archival application</summary>
<pre>
</pre>
</details>
	
<details><summary>Automating effort to get expected counts from sql scripts and comparing it to final counts in cloud application for chain of custody -  Example Included</summary>
<pre>				
if useTable:
	print('\nChecking Chain of Custody for file: {}'.format(file))
	delete = sqlDict.get('delete','')
	whereSplit = re.split( 'WHERE', delete, flags=re. IGNORECASE)
	whereStr =
	if len(whereSplit) › 1:
		whereStr 'WHERE ' + whereSplit[1]
	archCountSQL = 'SELECT COUNT (*) FROM {} {}' .format(fullTable, wherestr)
	print(archCountSQL.strip()‚end=': ')
	archCount = list(cursor .execute(archCountSQL))[0][0]
	print(archCount)
	diff = None
	sourceCount = None
	if not archieOnly:
		ouputFilename = os.path.join(SQLFoldPath, 'Count For '+baseFile)
		cDict = cteDict['dict']
	
		if co. SOLTYPE == 'Warehouse':
			if cDict:
				print('Creating {} Temp Table (s)'.format(len(cDict)))


				try:
					runStringQuery(co.ARCHIECNXNSTRING, sqlDict['drop' ]['string'],ret=False, useCurs=False)
					runStringQuery(co.ARCHIECNXNSTRING, cteDict[ 'string'], ret=False , useCurs=False) 
				except Exception as e:
					print(e)
			cDict = {}
		print('Getting counts from each segment in file:' ‚end=' ')
		sourceCount = self.getItemCounts (cursor, select, cDict, variables, ouputFilename)
		print(sourceCount)
		diff = sourceCount - archCount print('Difference: '‚diff)
		
		retDict = [{'CheckTime':str(runTime)[:19], 'CheckID' :runID, 'FromFile': file, 'TableName' : table, 'SchemaName' : schema, 'ArchieDatabase':co.ARCHIEDATABASE, 'SourceCount': sourceCount, 'ArchieCount' : archCount, 'Difference': diff, 'WhereString' :whereStr}]
		resTables += [{'title':basefile, 'table' :retDict, 'subtitle':message}] 
		insertTableDict(co.ARCHIECNXNSTRING, retDict, 'ChainofCustody' ‚sch='dora')
	else:
		message = 'No Archie Table found for file {} in the script or filename. Filenames should look like "ItemSchema.ItemName - ItemSubType.sql"'.format(file)
		print (message)
		resTables += ['title' :baseFile, 'table' :None, 'subtitle': message}]

def getItemCounts (self, crsr, selectStatement, cteDict,variables, fn=''):
	try:
		segs = ps.splitSelSegments (selectStatement)
	except:
		print (selectStatement)
		segs = []
	total = 0
	sqlStringList = []
	for seg in segs:
		segDict = ps.cleanSegment(seg)
		segCount, segStr = ps.getSegCount(crsr, segDict, cteDict, variables)
		total += segCount
		segStr = re.sub('[\n]+', '\n' , segStr)
		sqlStringList.append(segStr)
	sqlstring = variables + '\n UNION \n'. join(sqlstringList)
	if fn:
		file = open (fn, 'w')
		file.write(sqlstring)
		file.close ()
	return total
				
</pre>
</details>
				
Some things that I would do differently given more resources and my current experience:
1. Use Django (Especially ORM) did not see the power of an ORM when I started building dora so I bascially built my own functions to interact with the database. It was a good learning experience but would never do it again.
2. Take time to refactor. After code was written and functional, I rarely had time to go back and refactor unless something broke or was performing too slowly
3. Break things down into smaller functions. Most of the functions i wrote are way too many lines of code, goes back to the lack to code review and time to refactor
4. Use of more classes as opposed to dictionaries for passing data between interfaces
5. The application ran on a local machine or customer VM, ideally would have used [Azure](https://learn.microsoft.com/en-us/azure/app-service/quickstart-python?tabs=flask%2Cwindows%2Cazure-cli%2Cvscode-deploy%2Cdeploy-instructions-azportal%2Cterminal-bash%2Cdeploy-instructions-zip-azcli 
) so it would live where the data was being transformed and run more performantly, with less setup from end users
			

