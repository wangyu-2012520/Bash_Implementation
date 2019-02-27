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
