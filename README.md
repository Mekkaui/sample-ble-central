BLE Central IoT Companion App
=============================
See [LICENSE.md](LICENSE.md) for license terms and conditions.

A sample application that demonstrates how to scan, connect, read and write data to a central from a Bluetooth Low Energy (BLE) peripheral via a service and it's specific characteristic.

###Scan for all BLE peripherals with or without a Service UUID
```javascript
/*Description: Add discovered device to tempArray*/
	var onDiscoverDevice = function(device) {
	    console.log(JSON.stringify(device));
	 	//Add device to array
	 	tempArray.push(device);
	};

	/*Description: Set device list and make visible*/
	var setDeviceList = function(){
		$interval.cancel(timeReduction)
		$scope.scanStatus.phase = "complete";
		$scope.scanStatus.text = "Scan Complete";
		$scope.blePeripherals = tempArray;
		angular.element(document.querySelectorAll("#peripheral-list")).removeClass("hidden");
	}

	/*Description: Scan for all BLE peripheral devices or for devices with a specific service UUID*/
	$scope.scan = function(serviceId) {
		seconds = 5;
		$scope.scanStatus.phase = "in-progress";
		$scope.scanStatus.text = "Scanning for devices....(5s left)";
		tempArray = [];
		angular.element(document.querySelectorAll("#peripheral-list")).addClass("hidden");
		if(serviceId !== undefined){
			if((serviceId.length !== 0) || (serviceId.replace(/\s/g, '').length)) { //Not spaces or empty
				console.log("Scan for Service UUID "+serviceId);
				blePerpheralsService.setServiceId(serviceId);
				//BLE Cordova plugin
				ble.scan([serviceId.toUpperCase()], seconds, onDiscoverDevice, blePerpheralsService.onError);
			}
		}
		else {
			console.log("Scan for all");
			//BLE Cordova plugin
			ble.scan([], seconds, onDiscoverDevice, blePerpheralsService.onError);
		}
		$scope.isScanBtnDisabled = true;

		var secondsLeft = seconds;

		//Set interval to visibly display the remaining seconds
		var timeReduction = $interval(function() {
			secondsLeft = secondsLeft - 1;
			if(secondsLeft > 0) {
				$scope.scanStatus.text = "Scanning for devices....(" + secondsLeft + "s left)";
			}
		}, 1000)

		//Set Timeout for applying device info to cards
		$timeout(setDeviceList, (seconds*1000));

	}

	/*Description: Connect to BLE peripheral*/
	$scope.connect = function(id, name) {
		blePerpheralsService.setSelectedDeviceId(id);
		blePerpheralsService.setSelectedDeviceName(name);
		var onConnect = function() {
			$scope.navigateTo("connection");
		}
		ble.connect(blePerpheralsService.getSelectedDeviceId(), onConnect, blePerpheralsService.onError);
	}

	/*Description: Transition to desired state/View*/
	$scope.navigateTo = function(stateName) {
		$state.go(stateName);
	}

})
```

###Read Data from connected BLE peripheral
```javascript
/*onRead callback*/
	var onRead = function(data) {
		console.log("data read");
        var str = bytesToString(data);
        console.log(str);
        //Add str to element on connection state/view
        $scope.$apply(function(){ //Access scope from outside function
        	$scope.dataMessage = str;
        });
	}

	/*Description: Read Data from a BLE connected peripheral*/
	$scope.readData = function() {
        //read(device_id, service_uuid, characteristic_uuid, success_function, failure_function)
        ble.read(blePerpheralsService.getSelectedDeviceId(), blePerpheralsService.getServiceId(), blePerpheralsService.getCharacteristicId(), onRead, blePerpheralsService.onError);
    }
```
###Write Data to a connected BLE peripheral
```javascript
/*onWrite callback*/
	var onWrite = function() {
	    console.log("data written to BLE peripheral");
	}

    /*Description: Write data to BLE peripheral*/
    $scope.writeData = function() {
        console.log("writeData");
        var str = "Hello from mobile device!!!";
        console.log(stringToBytes(str));
        ble.write(blePerpheralsService.getSelectedDeviceId(), blePerpheralsService.getServiceId(), blePerpheralsService.getCharacteristicId(), stringToBytes(str), onWrite, blePerpheralsService.onError);
    }
```

###Disconnect from BLE peripheral
```javascript
ble.disconnect(blePerpheralsService.getSelectedDeviceId(), backToHome, blePerpheralsService.onError);

```

Important App Files
---------------------------
* index.html
* index.css
* index.js
* screenshot.png
* app.json
* README.md