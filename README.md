# Installing GHC iOS

* Prerequisites:
   * Install Xcode http://itunes.apple.com/us/app/xcode/id497799835
   * Update Cabal with ```cabal update && cabal install cabal-install```
      * Remove **jobs: $ncpus** from *~/.cabal/config* if it's there
   * Download these scripts and place them in your PATH. For example:
```
git clone git@github.com:ghc-ios/ghc-ios-scripts.git ~/bin/ghc-ios-scripts
echo -e "\nPATH=~/bin/ghc-ios-scripts:"'$PATH' >> ~/.profile
PATH=~/bin/ghc-ios-scripts:$PATH
```

* Run the installer script. GHC iOS will politely live alongside your existing native GHC installation.
```
installGHCiOS.sh
```

# Using GHC iOS

* Create or open an Xcode project (the Single View Application template is simple for testing)
* Add HaskelliOS.xcconfig from this repository to your project. It doesn't need to be added to any targets.
    * Click on your project at the top of the Xcode sidebar, and make sure your project is selected within it.
    * In the "Info" tab, under "Configuration", expand each configuration and choose HaskelliOS.
    * You may need to adjust the HEADER_SEARCH_PATHS to match your installed GHC.
* Compile your Haskell code with ghc-ios. For example, if you had a file named Counter.hs:
```haskell
{-# LANGUAGE ForeignFunctionInterface #-}
module Counter where
import Control.Concurrent
import Control.Monad
foreign export ccall startCounter :: Int -> IO ()
startCounter :: Int -> IO ()
startCounter = void . forkIO . void . loop
    where loop i = do
            putStrLn (replicate i 'o')
            threadDelay (10^6)
            loop (i + 1)
```
* Compile this like so to get Counter.a and Counter_stub.h:
```
ghc-ios Counter
```
(Counter.a will be a fat binary that works with both devices and the simulator.)


* Drag Counter.a and Counter_stub.h to the project's sidebar. Make sure "Add to Targets:" has a check next to your app.

* In main.m ```#import "HsFFI.h"``` and add ```hs_init(NULL, NULL);``` to the top of ```main()```

* In your app's AppDelegate.m:
```
#import "Counter_stub.h"
```
* and at the top of *application:didFinishLaunchingWithOptions:*, add:
```
    startCounter(3);
```

* Run your app! You should see a growing triangle of 'o's.

* To automatically rebuild your Haskell file each time you rebuild your project:
   * Click your project at the top of the Xcode sidebar, then click on your app under Targets on the left.
   * Click the "Build Phases" tab, then click the + at the top left and choose "New Run Script Build phase"
   * Drag the new Run Script build phase to the top of the list.
   * Enter:
```
source ~/.profile # or ~/.bash_profile, whichever you use
ghc-ios Counter
```
Xcode will now run these commands each time it builds your project.

# Using GHC iOS with Cabal

* You'll need a very recent version of Cabal: ```cabal install cabal-install```
* You'll also need to check that the option **jobs: $ncpus** does not appear in your *~/.cabal/config* file, as it triggers a mode that does not support cross-compilation.
* To install a package for the device and simulator, use cabal-ios (included in ghc-ios-scripts) like:
```
cabal-ios install text
```
* You should now be able to use the package in a file compiled with ghc-ios. The package will be statically linked into the .a library.

# Troubleshooting
* If you run into ```Illegal instruction: 4``` while running ```make install```, please see this discussion https://github.com/ghc-ios/ghc-ios-scripts/issues/4 — we're not sure what causes it yet but the thread contains a working solution.

