- Archive:
```
hadoop archive -archiveName results_2022_01.har -p ${path_to_data}/2022/01 ${path_to_archives}/2022 
```

- Verify:
```
hadoop fs -count ${path_to_data}/2022/01  
#and 
hadoop fs -count har:${path_to_archives}/2022/results_2022_01.har 
```