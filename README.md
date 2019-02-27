# Bash_Implementation
Bash_Implementation

## grep & sed
grep and sed can combine to do search string with customized regular express, here is my code to search for 'total test', 'pass', 'failed' ect
```
string=`docker logs ci_integration-test_1 2>&1 | grep "Total tests: [0-9]*. Passed: [0-9]*. Failed: [0-9]*. Skipped: [0-9]*."` #| sed -e 's/[^0-9 ]//g'`
TOTAL_TEST=`echo $string | grep "Total tests: [0-9]*." -o | sed 's/[^0-9]*//g'`
PASS=`echo $string | grep "Passed: [0-9]*." -o | sed 's/[^0-9]*//g'`
FAILED=`echo $string | grep "Failed: [0-9]*." -o | sed 's/[^0-9]*//g'`
IGNORE=`echo $string | grep "Skipped: [0-9]*." -o | sed 's/[^0-9]*//g'`
```

## example
here is the example for loop all docker-compose files, each docker-compose file contains a set of test scenarios; By doing this, we are able to put bash file (test scenarios only for functional test)as part of CI (or known as CI with test).

```
yourfilenames='./*.yml'  
for eachfile in $yourfilenames
do
	echo ${eachfile##*/} 
done

# define some colors to use for output
RED='\033[0;31m'
GREEN='\033[0;32m'
NC='\033[0m'
# kill and remove any running containers
cleanup () {
  docker-compose -p ci kill
  docker-compose -p ci rm -f
}

printlogs() {
	docker logs ci_integration-test_1
	docker logs ci_store-retriever_1
	docker logs ci_transit-generator_1
	docker logs ci_mssql_1
	docker logs ci_mock-server_1
}

getindividualresult() {
	string=`docker logs ci_integration-test_1 2>&1 | grep "Total tests: [0-9]*. Passed: [0-9]*. Failed: [0-9]*. Skipped: [0-9]*."` #| sed -e 's/[^0-9 ]//g'`
	TOTAL_TEST=`echo $string | grep "Total tests: [0-9]*." -o | sed 's/[^0-9]*//g'`
	PASS=`echo $string | grep "Passed: [0-9]*." -o | sed 's/[^0-9]*//g'`
	FAILED=`echo $string | grep "Failed: [0-9]*." -o | sed 's/[^0-9]*//g'`
	IGNORE=`echo $string | grep "Skipped: [0-9]*." -o | sed 's/[^0-9]*//g'`
}

summaryresult() {
	ALL_TEST_EXIT_CODE+=$TEST_EXIT_CODE
	ALL_TOTAL_TEST+=$TOTAL_TEST
	ALL_PASS+=$PASS
	ALL_FAILED+=$FAILED
	ALL_SKIPPED+=$SKIPPED
}

printresult() {
	printf "================================= Test Result =================================\n"
	printf "Testing exit code is ${ALL_TEST_EXIT_CODE}\n"
	printf "Total tests: ${ALL_TOTAL_TEST}\n"
	printf "Total Passed: ${ALL_PASS}\n"
	printf "Total Failed: ${ALL_FAILED}\n"
	printf "Total Skipped: ${ALL_SKIPPED}\n"
}
# catch unexpected failures, do cleanup and output an error message
trap 'cleanup ; printf "${RED}Tests Failed For Unexpected Reasons${NC}\n"'\
  HUP INT QUIT PIPE TERM

declare -i ALL_TEST_EXIT_CODE
declare -i TOTAL_TEST
declare -i PASS
declare -i FAILED
declare -i SKIPPED
declare -i ALL_TOTAL_TEST
declare -i ALL_PASS
declare -i ALL_FAILED
declare -i ALL_SKIPPED
ALL_TEST_EXIT_CODE=0
ALL_TOTAL_TEST=0
ALL_PASS=0
ALL_FAILED=0
ALL_SKIPPED=0

for eachfile in $yourfilenames
do  
	# build and run the composed services
	docker-compose -f ${eachfile##*/} -p ci build --no-cache && docker-compose -f ${eachfile##*/} -p ci up -d  

	if [ $? -ne 0 ] ; then
	  printf "${RED}Docker Compose Failed${NC}\n"
	  exit -1
	fi
	# wait for the test service to complete and grab the exit code
	TEST_EXIT_CODE=`docker wait ci_integration-test_1`
	
	printlogs
	getindividualresult
	summaryresult

	# inspect the output of the test and display respective message
	if [ -z ${TEST_EXIT_CODE+x} ] || [ "$TEST_EXIT_CODE" -ne 0 ] ; then
	  printf "${RED}Tests Failed${NC} - Exit Code: $TEST_EXIT_CODE\n"
	else
	  printf "${GREEN}Tests Passed${NC}\n"
	fi

	# call the cleanup fuction
	docker-compose -f ${eachfile##*/} -p ci kill
  	docker-compose -f ${eachfile##*/} -p ci rm -f
	# exit the script with the same code as the test service code
	# exit $TEST_EXIT_CODE
done

printresult
# exit $ALL_TEST_EXIT_CODE
```
