# UniSLuaPF
Plugin framework based on Unity+SLua
基于Unity和Slua的游戏开发框架

Features:

* Plugin Design.
* OPP Support.
* Security Code Enviroment.

* 插件模式设计，每个插件有独立的运行环境，彼此之间互不干扰。
* 支持面向对象。
* 安全的代码环境（例如不允许隐式定义全局变量，每个插件有自己独立的全局环境，无法在声明之外再为类增加字段或方法)

##Setup (初始化)
* SLua repository : https://github.com/pangweiwei/slua
* ShellHelper repository : https://github.com/wlgys8/UnityShellHelper
* LuaPlugins : user's code is here.  
* LuaFramwork : framework's code is here. 

1. Click Menu->Slua->UnityEngine->Make
2. Click Menu->Slua->Custom->Make
3. Add LuaManager.cs to a scene gameobject.
4. Set the `autoBoot` in inspector to `True` 

##How to add plugin (新建插件)

Create two folders named Plugin1 & Plugin2 under LuaPlugins.<br>
Then create a file named "main.lua" under each folder.

Structure：

    -LuaPlugins
    	-Plugin1
    		-main.lua
    	-Plugin2
    		-main.lua


1.Framework will traverse all folders under "LuaPlugins".<br>
2.Each one folder represents a plugin.<br>
3."main.lua" is used as the entry file.<br>

1.框架会遍历LuaPlugins目录下的文件夹<br>
2.每个文件夹代表一个插件<br>
3."main.lua" 作为插件的执行入口<br>


### main.lua 

		plugin{
			name = 'Plugin1', --plugin name
			main = function ( ... ) --entry function
			end,
			dependencies = { -- depend on any other plugins?
				"Plugin2",
			}
		}
##How to share code between plugins (插件之间的互相访问)

##How to build plugins as assetBundles (如何打包成AssetsBundle)
Click Menu->Build->BuildLuaPlugins

Framework will generate assetbundles under Assets/Output/\<Platform\>/

##How to compile lua to bytecodes (如何将Lua编译成bytecode)
* click menu->build->config
* set compiler path for your platform.
* click menu->build->buildLuaPlugins

either luac or luajit is ok.

important:the vesion of lua or luajit used for compiling must be the same as the lib's version which under the folder "Plugin/\<Platform\>"

Builder will search by path in order until get an available compiler.
If no compiler is found, then the builder won't compile lua scripts.It will only change the extension from "\*.lua" to "\*.lua.txt"

##How to load lua from assetbundles. (如何从AssetBundle里加载代码）

1. Put assetbundles built above under StreamingAssets
2. Set LuaManager.mode = LuaRunMode.AssetBundle.

In Editor mode,luaManager will load lua file directly from editor,all modifications on files will be work immediately.This mode is recommanded when you are in developing.

In AssetBundle mode,luaManager will load lua file from assetbundle.If some files get changed,they should be rebuilt to to make the modifications work.

##If my assetbundles are on the server? （如何从服务器上下载插件代码)
Set LuaManager.autoBoot = false,and boot the lua manager by yourself.

Before call LuaManager.LoadAllPlugins(), assign the url to the assetbundle that you want to load from server.

		IEnumerator Start(){
			yield return manager.Setup();
			manager.bundleManager.AddURL("bundleName","http://url-for-bundle");
			.
			.
			.
			yield return manager.LoadAllPlugins();
			yield return manager.LaunchPlugin();
		
		}

manager will load assetbundles from Application.StreamingAssets if there are no urls assigned to them.
		

		
##OOP implements (面向对象实现)

We can use keywork 'class' to define a class:

实现了class关键字.可以如下定义一个类型:

    class('ClassA')

	--static field
	staticVar = 1

	--private static field
	local pStaticVar = 2

	-- construct function
	function ClassA:ctor(...)
		-- member field
		self.memberVar = 3
		print 'call class A ctor'
	end

	--member function
	function ClassA:foo( ... )
		print 'call class A member function foo'
	end

	--static function
	function ClassA.foo2( ... )
		print 'call classAstatic function foo2'
	end

	--also static function
	function foo3( ... )
		print 'call classA static function foo3'
	end

	-- private static function
	local function foo4( ... )
	end

	classend()
	
	ClassA.newfiled = 1 --this will throw an error,cause we can not define new field or function any more after 'classend()'

the way to use:	

调用方式:

    local a = ClassA()
    a:foo()

##How to receive MonoBehaviour Messages?
Use LBehaviour.

LBehaviour works like MonoBehaviour in lua. It can be added to gameObject and receive MonoBehaviour messages.

	class('MyBehaviour' ,LBehaviour)

		function MyBehaviour:Start()
			print('Start')
		end
		
		function MyBehaviour:OnEnable ()
			print('on enable')
		end

		function MyBehaviour:OnDisable()
			print('on disable')
		end

		function MyBehaviour:OnDestroy (  )
			print('on destroy')
		end


	classend()

	LBehaviour.AddTo(gameObject,MyBehaviour)
