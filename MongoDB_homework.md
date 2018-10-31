
var map1 = function() {
	for (var idx = 0; idx < this.items.length; idx++) {
	   var month = this.items[idx].date.substring(0, 7);
	   var x = Math.floor((this.items[idx].weidu - 3.86) / 10);
	   var y = Math.floor((this.items[idx].jingdi - 73.66) / 10);
	   var key = month+"@"+x+y;
	   var value = {
					 count: 1,
					 deep_value: this.items[idx].deep_value
				   };
	   emit(key, value);
	}
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
	

db.collection.mapReduce(
   map1,  
   reduce1,   
   {
      out: { merge: "eq" },
      query: { $and: [ this.jingdi:{$ge:73.66}, this.jingdi:{$le:135.05}, this.weidu:{$ge:3.86}, this.weidu:{$le:53.55} ] },
   }
);
