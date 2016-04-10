# Bluetooth ANE V2.0 (Android)
With Bluetooth ANE, you'll have access to the Bluetooth hardware. It enable you to scan for other devices, connect to and pair with them and finally transfer data between them. This ANE has been built with consideration of connecting to Android devices to each other. if you need it for other purposes, feel free to contact us here: http://www.myflashlabs.com/contact/ (This is NOT a Bluetooth LE API)

# Demo .apk
you may like to see the ANE in action? [Download demo .apk](https://github.com/myflashlab/bluetooth-ANE/tree/master/FD/dist)

**NOTICE**: the demo ANE works only after you hit the "OK" button in the dialog which opens. in your tests make sure that you are NOT calling other ANE methods prior to hitting the "OK" button.
[Download the ANE](https://github.com/myflashlab/bluetooth-ANE/tree/master/FD/lib)

# Air Usage:
```actionscript
package 
{
	import com.myflashlab.air.extensions.bluetooth.Bluetooth;
	import com.myflashlab.air.extensions.bluetooth.BluetoothEvent;
	import com.doitflash.consts.Direction;
	import com.doitflash.consts.Orientation;
	import com.doitflash.mobileProject.commonCpuSrc.DeviceInfo;
	import com.doitflash.starling.utils.list.List;
	import com.doitflash.text.modules.MySprite;
	import com.greensock.TweenMax;
	import flash.desktop.NativeApplication;
	import flash.desktop.SystemIdleMode;
	import flash.display.Sprite;
	import flash.display.StageAlign;
	import flash.display.StageScaleMode;
	import flash.events.Event;
	import flash.events.InvokeEvent;
	import flash.events.KeyboardEvent;
	import flash.events.MouseEvent;
	import flash.text.AntiAliasType;
	import flash.text.TextField;
	import flash.text.TextFieldAutoSize;
	import flash.text.TextFieldType;
	import flash.text.TextFormat;
	import flash.text.TextFormatAlign;
	import flash.ui.Keyboard;
	import flash.ui.Multitouch;
	import flash.ui.MultitouchInputMode;
	
	/**
	 * ...
	 * @author Hadi Tavakoli - 2/24/2014 5:30 PM
	 */
	public class Demo2 extends Sprite 
	{
		private var _ex:Bluetooth;
		private var _buttonsHolder:Sprite;
		private var _devicesHolder:Sprite;
		private var _list:List;
		
		private var _txt:TextField;
		
		private var _ball:MySprite;
		
		private const BTN_WIDTH:Number = 150;
		private const BTN_HEIGHT:Number = 100;
		
		public function Demo2():void 
		{
			Multitouch.inputMode = MultitouchInputMode.GESTURE;
			NativeApplication.nativeApplication.addEventListener(Event.ACTIVATE, handleActivate, false, 0, true);
			NativeApplication.nativeApplication.addEventListener(Event.DEACTIVATE, handleDeactivate, false, 0, true);
			NativeApplication.nativeApplication.addEventListener(InvokeEvent.INVOKE, onInvoke, false, 0, true);
			NativeApplication.nativeApplication.addEventListener(KeyboardEvent.KEY_DOWN, handleKeys, false, 0, true);
			
			stage.addEventListener(Event.RESIZE, onResize);
			stage.scaleMode = StageScaleMode.NO_SCALE;
			stage.align = StageAlign.TOP_LEFT;
			
			_buttonsHolder = new Sprite();
			_buttonsHolder.visible = false;
			this.addChild(_buttonsHolder);
			
			_devicesHolder = new Sprite();
			_devicesHolder.visible = false;
			this.addChild(_devicesHolder);
			
			_list = new List();
			_list.holder = _devicesHolder;
			_list.itemsHolder = new Sprite();
			_list.orientation = Orientation.VERTICAL;
			_list.hDirection = Direction.LEFT_TO_RIGHT;
			_list.vDirection = Direction.TOP_TO_BOTTOM;
			_list.space = 2;
			
			init();
			onResize();
		}
		
		private function onInvoke(e:InvokeEvent):void
		{
			NativeApplication.nativeApplication.removeEventListener(InvokeEvent.INVOKE, onInvoke);
		}
		
		private function handleActivate(e:Event):void
		{
			NativeApplication.nativeApplication.systemIdleMode = SystemIdleMode.KEEP_AWAKE;
		}
		
		private function handleDeactivate(e:Event):void
		{
			
		}
		
		private function handleKeys(e:KeyboardEvent):void
		{
			if(e.keyCode == Keyboard.BACK)
            {
				if (_ex) 
				{
					_ex.dispose(); // ALWAYS dispose the bluetooth extension before closing your app
				}
				e.preventDefault();
				NativeApplication.nativeApplication.exit();
            }
		}
		
		private function onResize(e:*=null):void
		{			
			if (_devicesHolder)
			{
				_devicesHolder.y = _buttonsHolder.y + _buttonsHolder.height + 5;
			}
			
			if (_txt)
			{
				_txt.width = stage.stageWidth;
				_txt.height = 100;
				_txt.y = stage.stageHeight - _txt.height;
				_txt.x = stage.stageWidth / 2 - _txt.width / 2;
			}
		}
		
		private function init():void
		{
			_txt = new TextField();
			_txt.mouseEnabled = false;
			_txt.selectable = false;
			//_txt.autoSize = TextFieldAutoSize.LEFT;
			_txt.defaultTextFormat = new TextFormat(null, 22, null, null, null, null, null, null, "center");
			_txt.text = "Scan to find a device";
			_txt.scaleX = _txt.scaleY = DeviceInfo.dpiScaleMultiplier;
			this.addChild(_txt);
			
			// initialize the extension
			_ex = new Bluetooth();
			_ex.addEventListener(BluetoothEvent.BLUETOOTH_STATE , bluetoothState);
			_ex.addEventListener(BluetoothEvent.COMMUNICATION_STATUS , communication);
			_ex.addEventListener(BluetoothEvent.CONNECTION , connection);
			_ex.addEventListener(BluetoothEvent.DIALOG_STATUS , dialog);
			_ex.addEventListener(BluetoothEvent.DISCOVERING_STATUS , discovering);
			_ex.addEventListener(BluetoothEvent.NEW_DISCOVERD , newDiscoverd);
			_ex.addEventListener(BluetoothEvent.READ_MESSAGE , readMessage);
			_ex.addEventListener(BluetoothEvent.SCAN_MODE , scanMode);
			_ex.addEventListener(BluetoothEvent.SEND_MESSAGE , sendMessage);
			
			var btn1:MySprite = createBtn("start scan");
			btn1.addEventListener(MouseEvent.CLICK, startScan);
			_buttonsHolder.addChild(btn1);
			
			function startScan(e:MouseEvent):void
			{
				_ex.startDiscovery();
			}
			
			// check if bluetooth is on
			if (_ex.isEnable)
			{
				_buttonsHolder.visible = true;
				_devicesHolder.visible = true;
				_ex.visible(0);
				_ex.initCommunicationService();
			}
			else
			{
				_ex.enable();
			}
		}
		
		private function initBall():void
		{
			if (_ball) removeBall();
			
			_list.removeAll();
			
			_ball = new MySprite();
			_ball.addEventListener(MouseEvent.MOUSE_DOWN, onDown);
			_ball.bgColor = 0xdf4c47;
			_ball.bgAlpha = 1;
			_ball.bgBottomLeftRadius = 20;
			_ball.bgBottomRightRadius = 20;
			_ball.bgTopLeftRadius = 20;
			_ball.bgTopRightRadius = 20;
			_ball.width = 200;
			_ball.height = 200;
			_ball.drawBg();
			_ball.x = stage.stageWidth / 2 - _ball.width / 2;
			_ball.y = stage.stageHeight / 2 - _ball.height / 2;
			this.addChild(_ball);
			TweenMax.from(_ball, 0.5, {alpha:0} );
			
			var txt:TextField = new TextField();
			txt.mouseEnabled = false;
			txt.selectable = false;
			txt.autoSize = TextFieldAutoSize.LEFT;
			txt.multiline = true;
			txt.defaultTextFormat = new TextFormat(null, 30, 0xFFFFFF, null, null, null, null, null, "center");
			txt.htmlText = "DRAG<br>ME";
			txt.scaleX = _txt.scaleY = DeviceInfo.dpiScaleMultiplier;
			txt.x = _ball.width / 2 - txt.width / 2;
			txt.y = _ball.height / 2 - txt.height / 2;
			_ball.addChild(txt);
		}
		
		private function removeBall():void
		{
			this.removeChild(_ball);
			_ball = null
		}
		
		private function onDown(e:MouseEvent):void
		{
			_ball.startDrag(false);
			_ball.addEventListener(MouseEvent.MOUSE_MOVE, onMove);
			_ball.addEventListener(MouseEvent.MOUSE_UP, onUp);
		}
		
		private function onMove(e:MouseEvent):void
		{
			//trace(_ball.x, _ball.y)
			_ex.sendMessage(String(Number(_ball.x / stage.stageWidth).toFixed(2)) + "|" + String(Number(_ball.y / stage.stageHeight).toFixed(2)));
		}
		
		private function onUp(e:MouseEvent):void
		{
			_ball.stopDrag();
			_ball.removeEventListener(MouseEvent.MOUSE_MOVE, onMove);
			_ball.removeEventListener(MouseEvent.MOUSE_UP, onUp);
		}
		
		private function bluetoothState(e:BluetoothEvent):void
		{
			trace("state >> " + e.param);
			
			if (e.param == "bluetoothOn")
			{
				_buttonsHolder.visible = true;
				_devicesHolder.visible = true;
				_ex.visible(0);
				_ex.initCommunicationService();
			}
		}
		
		private function communication(e:BluetoothEvent):void
		{
			trace("communication >> " + e.param);
			
			if (e.param == "connecting")
			{
				_txt.text = "connecting to the device...";
			}
			else if (e.param == "connected")
			{
				_txt.text = "CONNECTED";
				initBall();
			}
		}
		
		private function connection(e:BluetoothEvent):void
		{
			trace("connection >> " + e.param);
			
			if (e.param == "connectionLost")
			{
				_txt.text = "connection lost!";
				removeBall();
			}
		}
		
		private function dialog(e:BluetoothEvent):void
		{
			trace("dialogStatus >> " + e.param);
		}
		
		private function discovering(e:BluetoothEvent):void
		{
			trace("discovering >> " + e.param);
			
			if (e.param == "discoveringStarted")
			{
				_txt.text = "Scaning...";
				
				// remove everything inside _devicesHolder
				_list.removeAll()
				
				_devicesHolder.alpha = 0.5;
				_devicesHolder.mouseEnabled = false;
				_devicesHolder.mouseChildren = false;
			}
			else if (e.param == "discoveringFinished")
			{
				_txt.text = "Scan finished";
				
				_devicesHolder.alpha = 1;
				_devicesHolder.mouseEnabled = true;
				_devicesHolder.mouseChildren = true;
			}
		}
		
		private function newDiscoverd(e:BluetoothEvent):void
		{
			var obj:Object = e.param;
			
			trace("newDiscoverd >> " + "device = " + obj.device + " mac :" + obj.mac);
			
			var btn1:MySprite = createBtn("Device: " + obj.device);
			btn1.data = obj;
			btn1.addEventListener(MouseEvent.CLICK, toConnect);
			_list.add(btn1);
			_list.itemArrange();
			
			function toConnect(e:MouseEvent):void
			{
				var mac:String = e.target["data"].mac;
				
				// check if we are paired?
				var pairList:Array = _ex.pairedList;
				var currDevice:Object;
				for (var i:int = 0; i < pairList.length; i++) 
				{
					currDevice = pairList[i];
					/*for (var name:String in currDevice) 
					{
						trace(name + " = " + currDevice[name])
					}
					trace("-----------")*/
					if (currDevice.mac == mac)
					{
						_ex.connectTo(mac);
						return;
					}
				}
				
				// if we are here, it means that the devices are not paired. let's pair them now!
				_ex.pairWith(mac);
			}
		}
		
		private function readMessage(e:BluetoothEvent):void
		{
			trace("readMessage >> " + e.param);
			var arr:Array = e.param.split("|");
			
			// bluetooth speed is so high sometimes that it sends two pair of ball points!
			if (arr.length > 2) return; // validate the incoming value to make sure we have one X and one Y value
			
			TweenMax.to(_ball, 0.3, {x:stage.stageWidth * Number(arr[0]), y:stage.stageHeight * Number(arr[1])} );
			//_ball.x = stage.stageWidth * Number(arr[0]);
			//_ball.y = stage.stageHeight * Number(arr[1]);
		}
		
		private function scanMode(e:BluetoothEvent):void
		{
			trace("scanMode >> " + e.param);
		}
		
		private function sendMessage(e:BluetoothEvent):void
		{
			trace("sendMessage >> " + e.param);
		}
		
		private function createBtn($str:String, $txtColor:uint=0x666666, $editable:Boolean=false):MySprite
		{
			var sp:MySprite = new MySprite();
			sp.addEventListener(MouseEvent.MOUSE_OVER,  onOver);
			sp.addEventListener(MouseEvent.MOUSE_OUT,  onOut);
			sp.addEventListener(MouseEvent.CLICK,  onOut);
			sp.bgAlpha = 1;
			sp.bgColor = 0xDFE4FF;
			sp.drawBg();
			sp.width = BTN_WIDTH * DeviceInfo.dpiScaleMultiplier;
			sp.height = BTN_HEIGHT * DeviceInfo.dpiScaleMultiplier;
			
			function onOver(e:MouseEvent):void
			{
				sp.bgAlpha = 1;
				sp.bgColor = 0xFFDB48;
				sp.drawBg();
			}
			
			function onOut(e:MouseEvent):void
			{
				sp.bgAlpha = 1;
				sp.bgColor = 0xDFE4FF;
				sp.drawBg();
			}
			
			var format:TextFormat = new TextFormat("Arimo", 18, $txtColor, null, null, null, null, null, TextFormatAlign.CENTER);
			
			var txt:TextField = new TextField();
			txt.autoSize = TextFieldAutoSize.LEFT;
			txt.antiAliasType = AntiAliasType.ADVANCED;
			txt.mouseEnabled = $editable;
			txt.multiline = true;
			txt.wordWrap = true;
			txt.border = $editable;
			if ($editable) txt.type = TextFieldType.INPUT;
			txt.scaleX = txt.scaleY = DeviceInfo.dpiScaleMultiplier;
			txt.width = sp.width * (1 / (DeviceInfo.dpiScaleMultiplier));
			txt.defaultTextFormat = format;
			txt.htmlText = $str;
			
			txt.y = sp.height - txt.height >> 1;
			sp.addChild(txt);
			sp.data.txt = txt;
			
			return sp;
		}
	}
	
}
```

# Air .xml manifest
```xml
<uses-permission android:name="android.permission.BLUETOOTH" />
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />

<!-- 	
	Required for Bluetooth to work properly from Android 6.0 or higher. More info here: 
	http://developer.android.com/about/versions/marshmallow/android-6.0-changes.html#behavior-hardware-id 
-->
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
```

# Requirements:
* Android SDK 10 or higher
* Air SDK 19 or higher

# Commercial Version
http://www.myflashlabs.com/product/bluetooth-ane-adobe-air-native-extension/

![Bluetooth ANE](http://www.myflashlabs.com/wp-content/uploads/2015/11/product_adobe-air-ane-extension-bluetooth-595x738.jpg)

# Tutorials
[How to embed ANEs into **FlashBuilder**, **FlashCC** and **FlashDevelop**](https://www.youtube.com/watch?v=Oubsb_3F3ec&list=PL_mmSjScdnxnSDTMYb1iDX4LemhIJrt1O)  
[Step by step tutorial showing you how to use this ANE to make a multiplayr-game](http://www.myflashlabs.com/build-multiplayer-games-in-adobe-air-using-bluetooth-ane/)  

# Changelog
*Jan 01, 2016 - V2.0*
* upgraded to Android Studio


*Nov 02, 2015 - V1.9*
* removed android-support-v4.jar dependency
* doitflash devs merged into MyFLashLab Team


*Feb 14, 2015 - V1.5*
* added Android 64 support


*Feb 25, 2014 - V1.0*
* beginning of the journey!