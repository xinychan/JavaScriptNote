# 流程控制

## 条件语句

if 语句

    // if 判断
    var exp = true;
    if (exp) {
    	document.write("exp is true");
    }
    if (exp) {
    	document.write("exp is true");
    } else {
    	document.write("exp is false");
    }

switch 语句

    // switch 判断使用 === 进行比较
    var index = 1;
    switch (index) {
    	case 0:
    		document.write("index is 0");
    		break;
    	case 1:
    		document.write("index is number 1");
    		break;
    	case "1":
    		document.write("index is string 1");
    		break;
    	default:
    		document.write("index default");
    		break;
    }

## 循环语句

for 循环

    for (var i = 0; i < 10; i++) {
    	document.write("index = " + i);
    	document.write("<br />");
    }

while 循环

    var index = 5;
    while (index > 0) {
    	document.write("index = " + index);
    	document.write("<br />");
    	index--;
    }

do while 循环

    var index = 1;
    do {
    	document.write("do action");
    	document.write("<br />");
    	index++;
    } while (index < 0)

## continue/break

continue 跳出当前循环条件

	for (var i = 0; i < 10; i++) {
		if (i == 5) {
            // 仅跳过等于5的条件
			continue;
		}
		document.write("index = " + i);
		document.write("<br />");
	}

break 跳出当前循环体

	for (var i = 0; i < 10; i++) {
		if (i == 5) {
            // 当条件等于5时跳出循环体
			break;
		}
		document.write("index = " + i);
		document.write("<br />");
	}