import groovy.json.JsonSlurper


@NonCPS
def jsonParse(def json) {
    new groovy.json.JsonSlurper().parseText(json)
}

def toJSON(def json) {
    new groovy.json.JsonOutput().toJson(json)
}

String CONFIGDETAILS = ""
String ARN = ""
String INSTANCEALIAS = ""
String ENABLEINBOUNDCALLS = ""
String ENABLEOUTBOUNDCALLS = ""
String IDENTITYMANAGEMENTTYPE = ""
String APPROVEDORIGINS = ""
String APPROVEDLAMBDAS = ""
String APPROVEDLEXBOTS = ""
String CONTACTFLOWLOGS = ""
String CONTACTTRACERECORDS = ""
String AGENTEVENTS = ""
String CALLRECORDINGS = ""
String CHATTRANSCRIPTS = ""
String CHATATTACHMENTS = ""
String SCHEDULEDREPORTS = ""

pipeline {
    agent any
    stages {
      
        stage('Initialization') {
            steps {
                script{
                   try{
                      sh(script: "rm -r ac-instance-management", returnStdout: true)    
                   }catch (Exception e) {
                       echo 'Exception occurred: ' + e.toString()
                   }                   
                   sh(script: "git clone https://github.com/ramprasadsv/ac-instance-management.git", returnStdout: true)
                   sh(script: "ls -ltr", returnStatus: true)
                   CONFIGDETAILS = sh(script: 'cat parameters.json', returnStdout: true).trim()
                   def config = jsonParse(CONFIGDETAILS)
                    INSTANCEALIAS = config.instanceAlias
                    ENABLEINBOUNDCALLS = config.enableInboundCalls
                    ENABLEOUTBOUNDCALLS = config.enableOutboundCalls
                    IDENTITYMANAGEMENTTYPE = config.identityManagementType
                    APPROVEDORIGINS = config.approvedOrigins
                    APPROVEDLAMBDAS = config.approvedLambdas
                    APPROVEDLEXBOTS = config.approvedLexBots
                    CONTACTFLOWLOGS = config.contactFlowLogs
                    CONTACTTRACERECORDS = config.contactTraceRecords
                    AGENTEVENTS = config.agentEvents
                    CALLRECORDINGS = config.callRecordings
                    CHATTRANSCRIPTS = config.chatTranscripts
                    CHATATTACHMENTS = config.chatAttachments
                    SCHEDULEDREPORTS = config.scheduledReports                    
                }
            }
        }      
      
      
        stage('Create an Amazon Connect Instance'){
            steps {
                echo 'Creating the Amazon Connect Instance'
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
                    // List all the Buckets
                   script {
                      String inboundCallsEnabled = "" 
                      String outboundCallsEnabled = ""
                      if(ENABLEINBOUNDCALLS.equals("true")) {
                          inboundCallsEnabled = "--inbound-calls-enabled" 
                      }
                      if(ENABLEOUTBOUNDCALLS.equals("true")) {
                          outboundCallsEnabled = "--outbound-calls-enabled" 
                      }
                       
                      def parsedJson =  sh(script: "aws connect create-instance --identity-management-type CONNECT_MANAGED --instance-alias ${INSTANCEALIAS} ${inboundCallsEnabled} ${outboundCallsEnabled}", returnStdout: true).trim()
                      echo "Instance details : ${parsedJson}"
                      def instance = jsonParse(parsedJson)
                      echo "ARN : ${instance.Arn}" 
                      ARN = instance.Arn
                   }
                }
            }
        }
        stage('Check status of the Instance'){
            steps{
                echo 'Instance check'
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
                    script {
                      echo "Waiting for the instance status to become Active withing 5 minutes "
                      def count = 1
                      while(count <= 30) {
                            println "Sleeping for 10 seconds, iteration ${count}"
                            sleep(time:10,unit:"SECONDS")
                            count++
                            def di =  sh(script: "aws connect describe-instance --instance-id ${ARN}", returnStdout: true).trim()
                            echo "Instance Describe : ${di}"
                            def instanceStatus = jsonParse(di)
                            String status = "ACTIVE"
                            echo "Status : ${instanceStatus.Instance.InstanceStatus.equals(status)}"
                            if(instanceStatus.Instance.InstanceStatus.equals(status)){
                                println("Instance creation is completed")
                                count = 31    
                            }
                        }
                    }
                }
            }
        }
        
        stage('Approved Origins'){
            steps{
                echo 'Adding approved origings'
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
                    script {
                        def ao = APPROVEDORIGINS.split(",")
                        ao.each { obj ->
                            def di =  sh(script: "aws connect associate-approved-origin --instance-id ${ARN} --origin ${obj}", returnStdout: true).trim()
                            echo "Approved Origins : ${di}"
                        };
                    }
                }
            }
        }
        
        stage('Approved Lambda'){
            steps{
                echo 'Adding approved Lambdas'
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
                    script {
                        def ao = APPROVEDLAMBDAS.split(",")
                        ao.each { obj ->
                            def di =  sh(script: "aws connect associate-lambda-function --instance-id ${ARN} --function-arn ${obj}", returnStdout: true).trim()
                            echo "Approved Lambdas : ${di}"
                        };
                    }
                }
            }
        }
        
        stage('Approved Lexbot'){
            steps{
                echo 'Adding Approved Lexbots'
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
                    script {
                        def ao = APPROVEDLEXBOTS.split(",")
                        ao.each { obj ->
                            String[] str = obj.split(":") 
                            def region = str[0]
                            def lexBot = str[1]
                            def di =  sh(script: "aws connect associate-lex-bot --instance-id ${ARN} --lex-bot Name=${lexBot},LexRegion=${region}", returnStdout: true).trim()
                            echo "Instance LexBot : ${di}"                            
                        };
                    }
                }
            }
        }
        
        stage('Enable Contact Flow Logs'){
            steps{
                echo 'Enabling Contact Flow Logs'
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
                    script {                        
                        def di =  sh(script: "aws connect update-instance-attribute --instance-id ${ARN} --attribute-type CONTACTFLOW_LOGS --value ${CONTACTFLOWLOGS}", returnStdout: true).trim()
                        echo "Enable Contact Flow Logs : ${di}"
                    }
                }
            }
        }
        
        stage('Enable Contact Trace Records'){
            steps{
                echo 'Enabling CTRs into Firehose'
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
                    script {
                        def di =  sh(script: "aws connect associate-instance-storage-config --instance-id ${ARN} --resource-type CONTACT_TRACE_RECORDS --storage-config StorageType=KINESIS_FIREHOSE,KinesisFirehoseConfig={FirehoseArn=${CONTACTTRACERECORDS}}", returnStdout: true).trim()
                        echo "CTR : ${di}"
                    }
                }
            }
        }
        
        stage('Enable Agent Realtime Events'){
            steps{
                echo 'Enabling Agent Realtime events in Kinesis Data Streams'
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
                    script {
                        def di =  sh(script: "aws connect associate-instance-storage-config --instance-id ${ARN} --resource-type AGENT_EVENTS --storage-config StorageType=KINESIS_STREAM,KinesisStreamConfig={StreamArn=${AGENTEVENTS}}", returnStdout: true).trim()
                        echo "Agent Events : ${di}"
                    }
                }
            }
        }
        
        stage('Enable Call Recordings'){
            steps{
                echo 'Enabling call recordings into S3'
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
                    script {
                        String sc = CALLRECORDINGS
                        sc = sc.replaceAll('Instance_Alias', INSTANCEALIAS)
                        echo sc
                        def js = jsonParse(sc)
                        sc = "StorageType=S3"
                        //ssociationId=string,StorageType=string,S3Config={BucketName=string,BucketPrefix=string,EncryptionConfig={EncryptionType=string,KeyId=string}}
                        sc = sc.concat(",S3Config=\\{BucketName=").concat(js.S3Config.BucketName).concat(",BucketPrefix=").concat(js.S3Config.BucketPrefix)
                        sc = sc.concat(",EncryptionConfig=\\{EncryptionType=").concat(js.S3Config.EncryptionConfig.EncryptionType)
                        sc = sc.concat(",KeyId=").concat(js.S3Config.EncryptionConfig.KeyId).concat("\\}\\}")
                        echo sc
                        js = null
                        def di =  sh(script: "aws connect associate-instance-storage-config --instance-id ${ARN} --resource-type CALL_RECORDINGS --storage-config ${sc}", returnStdout: true).trim()
                        echo "Call Recordings : ${di}"
                    }
                }
            }
        }
        
        stage('Enable Chat Transcripts'){
            steps{
                echo 'Enabling chat transcripts into S3'
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
                    script {
                        def sc = CHATTRANSCRIPTS
                        sc = sc.replaceAll('Instance_Alias', INSTANCEALIAS)
                        echo sc
                        def js = jsonParse(sc)
                        sc = "StorageType=S3"
                        //ssociationId=string,StorageType=string,S3Config={BucketName=string,BucketPrefix=string,EncryptionConfig={EncryptionType=string,KeyId=string}}
                        sc = sc.concat(",S3Config=\\{BucketName=").concat(js.S3Config.BucketName).concat(",BucketPrefix=").concat(js.S3Config.BucketPrefix)
                        sc = sc.concat(",EncryptionConfig=\\{EncryptionType=").concat(js.S3Config.EncryptionConfig.EncryptionType)
                        sc = sc.concat(",KeyId=").concat(js.S3Config.EncryptionConfig.KeyId).concat("\\}\\}")
                        echo sc
                        js = null
                        def di =  sh(script: "aws connect associate-instance-storage-config --instance-id ${ARN} --resource-type CHAT_TRANSCRIPTS --storage-config ${sc}", returnStdout: true).trim()
                        echo "Chat Transcripts : ${di}"
                    }
                }
            }
        }
        
        stage('Enable Scheduled Reports'){
            steps{
                echo 'Enabling S3 for storing scheduled reports'
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
                    script {
                        def sc = SCHEDULEDREPORTS
                        sc = sc.replaceAll('Instance_Alias', INSTANCEALIAS)
                        echo sc
                        def js = jsonParse(sc)
                        sc = "StorageType=S3"
                        //ssociationId=string,StorageType=string,S3Config={BucketName=string,BucketPrefix=string,EncryptionConfig={EncryptionType=string,KeyId=string}}
                        sc = sc.concat(",S3Config=\\{BucketName=").concat(js.S3Config.BucketName).concat(",BucketPrefix=").concat(js.S3Config.BucketPrefix)
                        sc = sc.concat(",EncryptionConfig=\\{EncryptionType=").concat(js.S3Config.EncryptionConfig.EncryptionType)
                        sc = sc.concat(",KeyId=").concat(js.S3Config.EncryptionConfig.KeyId).concat("\\}\\}")
                        echo sc
                        js = null
                        def di =  sh(script: "aws connect associate-instance-storage-config --instance-id ${ARN} --resource-type SCHEDULED_REPORTS --storage-config ${sc}", returnStdout: true).trim()
                        echo "Chat Transcripts : ${di}"
                    }
                }
            }
        }

        /*stage('Enable Chat Attachments'){
            steps{
                echo 'Enabling S3 for storing chat attachments'
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
                    script {
                        def sc = CHATATTACHMENTS
                        sc = sc.replaceAll('Instance_Alias', INSTANCEALIAS)
                        echo sc
                        def js = jsonParse(sc)
                        sc = "StorageType=S3"
                        //ssociationId=string,StorageType=string,S3Config={BucketName=string,BucketPrefix=string,EncryptionConfig={EncryptionType=string,KeyId=string}}
                        sc = sc.concat(",S3Config=\\{BucketName=").concat(js.S3Config.BucketName).concat(",BucketPrefix=").concat(js.S3Config.BucketPrefix)
                        sc = sc.concat(",EncryptionConfig=\\{EncryptionType=").concat(js.S3Config.EncryptionConfig.EncryptionType)
                        sc = sc.concat(",KeyId=").concat(js.S3Config.EncryptionConfig.KeyId).concat("\\}\\}")
                        echo sc
                        js = null
                        def di =  sh(script: "aws connect associate-instance-storage-config --instance-id ${ARN} --resource-type CHAT_ATTACHMENTS --storage-config ${sc}", returnStdout: true).trim()
                        echo "Chat Transcripts : ${di}"
                    }
                }
            }
        }*/
        
    }
}
