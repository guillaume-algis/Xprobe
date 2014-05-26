## XprobePlugin - Objective-C App Memory Browser

![Icon](http://injectionforxcode.johnholdsworth.com/xprobe1.png)

The XprobePlugin is an Xcode extension that allows you to browse your application's memory in a
WebView inside Xcode. Build this project and restart Xcode to get up and running. When using the simulator
no change to your project is required and any time your application is running use the "Product/Xprobe/Load"
menu item to load the memory scanner into your app. It will preform an initial search of all objects referred
to in some way by a sweep of [UIApplication sharedApplication] and its windows. This initial
list of live objects is filtered by a class name regular expression to yield the "root"
objects displayed.

As a step toward auto-documenting applications, if you have "Graphviz" or "dot" installed on
your computer the set of swept objects can now be displayed as an directed graph showing all
objects which don't appear to be part of a "kit" (i.e. UIkit) and any objects they link to
("kit" or otherwise subject to an exclusion list.)

![Icon](http://injectionforxcode.johnholdsworth.com/xprobe2.png)

Clicking on one of these object links will drill down into viewing the ivars of the instance
and click on it's superclass link to view its ivars. Clicking on the link for an object ivar
value will browse to that object and so on. There are links to display the class' properties
and methods as extracted from the Objective-C runtime and a link to display subviews if possible.

The final link "trace" allows you to put a trace on the object such that all method calls will be
logged to the lower part of the Xprobe console along with their arguments. This can be filtred
in real time using a regular expression entered into the search field at the base. You need to
trace each class in the class' hierarchy separately to see absolutely all messages.

If you click on a simple value displayed for an ivar it can be edited by changing the text field
created and typing enter. Clicking on the name of properties or methods with no arguments the
value returned will be displayed. Method listings can now be searched which will also find
any methods matching inherited from superclasses. The "siblings" link will display links to
all objects sharing the same class. These are raw links to objects that where detected
at the time of the last sweep. Some objects may have since been deallocated so this may
crash alas. Siblings can also be found at the level of any shared superclasses.

### Use on a device.

Xprobe works by loading a bundle in the simulator which connects to Xcode when it is loaded.
An application makes its list of seed nodes known to Xprobe by implementing the following category:

    @implementation Xprobe(Seeding)

    + (NSArray *)xprobeSeeds {
        UIApplication *app = [UIApplication sharedApplication];
        NSMutableArray *seeds = [[app windows] mutableCopy];
        [roots insertObject:app atIndex:0];
        return seeds;
    }

    @end
    
Once an app is initialised call [Xprobe connectTo:"your.ip.address" retainObjects:YES] to
connect to the TCP server running inside Xcode. The retainObjects: argument specifies whether
to retain objects found in the sweep. This will make Xprobe more reliable but it will affect
object lifecyles in your app. After this, call [Xprobe search:@""] to perform the initial sweep 
starting at these objects looking for root objects. Each time "search:" is called or the object 
class filter is changed the sweep is performed anew. The application will need to be built with
Xtrace/{h,mm} to allow method tracing.

Re-iterating, after connecting, each time you search, Xprobe sweeps a set of seed objects to
find the set of all live objects that can be browsed. This list is than filtered according to
the class name pattern. An instance will be included if the name of one of it's superclasses
matches the pattern also. Some legacy classes are not "clean" and use "assign" properties
which may contain pointers to deallocated objects. To avoid sweeping the ivars of these
classes Xprobe has an exclusion filter which can be overridden (with a warning) in a category:

    @implementation Xprobe(ExclusionOverride)

    + (BOOL)xprobeExclude:(const char *)className {
        return className[0] == '_' || strncmp(className, "WebHistory", 10) == 0 ||
            strncmp(className, "NS", 2) == 0 || strncmp(className, "XC", 2) == 0 ||
            strncmp(className, "IDE", 3) == 0 || strncmp(className, "DVT", 3) == 0 ||
            strncmp(className, "Xcode3", 6) == 0 ||strncmp(className, "IB", 2) == 0;
    }
    
    @end
    
These exclusions allow Xprobe to work cleanly inside Xcode itself which comes in handy 
if you're a plugin dev. For any suggestions or feedback you can contact the author
on xprobe at johnholdsworth.com. Major releases will be announced on twitter
[@Injection4Xcode](https://twitter.com/#!/@Injection4Xcode).

### License

Copyright (c) 2014 John Holdsworth. Licensed for any use during development of Objective-C
applications, re-distribution may only be through github however including this copyright notice.

This release includes a very slightly modifed version of the excellent 
[canviz](https://code.google.com/p/canviz/) library to render "dot" 
files in an HTML canvas which is subject to an MIT license.
The change is to reverse the rendering of nodes and the lines
linking them on line 402 of "canviz-0.1/canviz.js".

### As ever:

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT 
LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. 
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, 
WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE 
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
