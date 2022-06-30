# Logging Ideas for Swift 

```swift
class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
        Logger.shared.debugPrint("Hello World")
        Logger.shared.prettyPrint("Hahaha")
        os_log("Hello", type: .error)
        do {
            let entries = try Logger.shared.getLogEntries()
            entries.forEach { log in
                print(log)
            }
        } catch {
            
        }
        
        
    }


}

// MARK: CUSTOM LOG 
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
    
    func getLogEntries() throws -> [OSLogEntryLog] {
        let subsystem = Bundle.main.bundleIdentifier!
        // Open the log store.
        let logStore = try OSLogStore(scope: .currentProcessIdentifier)

        let oneHourAgo = logStore.position(date: Date().addingTimeInterval(-3600))
        // Get all the logs from the last hour.
#if os(macOS)
    let allEntries = try logStore.getEntries(at: oneHourAgo)
#else
    // FB8518476: The Swift shims for the entries enumerator are missing.
    let allEntries = try Array(logStore.__entriesEnumerator(position: oneHourAgo, predicate: nil))
#endif


        // Filter the log to be relevant for our specific subsystem
        // and remove other elements (signposts, etc).
        
        return allEntries
            .compactMap { $0 as? OSLogEntryLog }
            .filter { $0.subsystem == subsystem }
    }
    
    func sendLogsToServer() {
            
    }
    
}

// MARK: - OS LOG EXTENSION
extension OSLog {
    private static var subsystem = Bundle.main.bundleIdentifier!
    
    static let ui = OSLog(subsystem: subsystem, category: "UI")
    static let network = OSLog(subsystem: subsystem, category: "Network")
    
}


// MARK: - WRITE LOG FILE
struct Log: TextOutputStream {

    func write(_ string: String) {
        let fm = FileManager.default
        let log = fm.urls(for: .documentDirectory, in: .userDomainMask)[0].appendingPathComponent("log.txt")
        if let handle = try? FileHandle(forWritingTo: log) {
            handle.seekToEndOfFile()
            handle.write(string.data(using: .utf8)!)
            handle.closeFile()
        } else {
            try? string.data(using: .utf8)?.write(to: log)
        }
    }
}

var logger = Log()
```
