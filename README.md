# Swift_CustomLogger

```swift

import UIKit

class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
        Logger.shared.debugPrint("Hello World")
        Logger.shared.prettyPrint("Hahaha")
    }


}

class Logger {
    
    static let shared = Logger()
    
    private init(){}
    
    func debugPrint(
        _ message: Any,
        extra1: String = #file,
        extra2: String = #function,
        extra3: Int = #line,
        remoteLog: Bool = false,
        plain: Bool = false
    ) {
        if plain {
            print(message)
        } else {
            let fileName: String = (extra1 as NSString).lastPathComponent
            print(message, "[\(fileName): Line \(extra3) of \(extra2)]")
        }
        
        // if remoteLog is true, record the log in server
        if remoteLog {
//            if let msg = message as? String {
//                logEvent(msg, event: .error, param: nil)
//            }
        }
        
    }
    
    /// pretty print
    func prettyPrint(_ message: Any) {
        dump(message)
    }
    
    func printDocumentsDirectory() {
        let documentsPath = NSSearchPathForDirectoriesInDomains(.documentDirectory, .userDomainMask, true)[0]
        print("Document Path: \(documentsPath)")
    }
    
    /// Track Event fo firebase
    func logEvent(_ name: String? = nil, event: String? = nil, param: [String: Any]? = nil) {
        // Analytics.logEvent(name, parameters: param)
    }
    
    
}


```
