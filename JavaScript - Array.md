Написать функцию contains. Если элементы массива what содержатся в массиве where, функция должна возвращать true
иначе false. Пустой массив является подмножеством любого массива.

```javascript
function contains(where, what) {
	 
  if (what.length === 0) {
  	
    return true;
  }
  
  for(var i = 0; i < what.length; i++){
  	
    if(where.indexOf(what[i]) === -1){
    		
        return false;
    }
  }
  return true;
}
```
