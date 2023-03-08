# personal

<details><summary>Test</summary>
<code>
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
  </code>
</details>

```
Collapsible Content2
```
