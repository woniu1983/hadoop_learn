```
# 导入数据
  > mongoimport --host ha11.woniu.com --port 30000 --db test --collection eq --type csv --headerline --ignoreBlanks --file /home/hadoop/download/data.csv

 # db.printShardingStatus()

---------------------------------------------------------------------------

var map1 = function() {
       var month = this.date.substring(0, 7);
       var x = Math.floor((this.weidu - 3.86) / 10);
       var y = Math.floor((this.jingdi - 73.66) / 10);
       var key = month+"@"+x+y;
       var value = {
                     count: 1,
                     deep_value: this.deep_value
                   };
       emit(key, value);
    
};
                    
var reduce1 = function(key, values) {

     var words = key.split("@");
     if (words.length <= 1) {
        return;
     }
     var monthVal = words[0];
     var addressGroupVal = words[1];
     reducedVal = {month : monthVal, addressGroup: addressGroupVal,  count: 0, lvls: [] };

     for (var idx = 0; idx < values.length; idx++) {
         reducedVal.count += values[idx].count;
         reducedVal.lvls.push(values[idx].deep_value);
     }

     return reducedVal;
};



db.eq.mapReduce(
   map1,  
   reduce1,   
   {
      out: { merge: "eq3" },
      query: { $and: [ {jingdi: { $gte : 73.66 }}, {jingdi: { $lte : 135.05 }}, {weidu: { $gte : 3.86 }}, {weidu: { $lte : 53.55 }} ] }
   }
);

```
