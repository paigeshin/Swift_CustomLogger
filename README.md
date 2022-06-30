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

# Viable Solution
```swift

//
//  ContentView.swift
//  OSLogStoreTesting
//
//  Created by bbruns on 23/12/2021.
//  Based on Peter Steinberger (23.08.20): https://github.com/steipete/OSLogTest/blob/master/LoggingTest/ContentView.swift.
//
import SwiftUI
import OSLog
import Combine

let subsystem = "com.bbruns.OSLogStoreTesting"

func getLogEntries() throws -> [OSLogEntryLog] {
    let logStore = try OSLogStore(scope: .currentProcessIdentifier)
    let oneHourAgo = logStore.position(date: Date().addingTimeInterval(-3600))
    let allEntries = try logStore.getEntries(at: oneHourAgo)

    return allEntries
        .compactMap { $0 as? OSLogEntryLog }
        .filter { $0.subsystem == subsystem }
}

struct SendableLog: Codable {
    let level: Int
    let date, subsystem, category, composedMessage: String
}

func sendLogs() {
    let logs = try! getLogEntries()
    let sendLogs: [SendableLog] = logs.map({ SendableLog(level: $0.level.rawValue,
                                                                 date: "\($0.date)",
                                                                 subsystem: $0.subsystem,
                                                                 category: $0.category,
                                                                 composedMessage: $0.composedMessage) })
    print(sendLogs)
    // Convert object to JSON
    let jsonData = try? JSONEncoder().encode(sendLogs)
    print(jsonData)
    
    // Send to my API
//    let url = URL(string: "http://x.x.x.x:8000")! // IP address and port of Python server
//    var request = URLRequest(url: url)
//    request.httpMethod = "POST"
//    request.httpBody = jsonData
//
//    let session = URLSession.shared
//    let task = session.dataTask(with: request) { (data, response, error) in
//        if let httpResponse = response as? HTTPURLResponse {
//            print(httpResponse.statusCode)
//        }
//    }
//    task.resume()
}

struct ContentView: View {
    let logger = Logger(subsystem: subsystem, category: "main")

    var logLevels = ["Default", "Info", "Debug", "Error", "Fault"]
    @State private var selectedLogLevel = 0

    init() {
        logger.log("SwiftUI is initializing the main ContentView")
    }

    var body: some View {
        return VStack {
            Text("This is a sample project to test the new logging features of iOS 15.")
                .padding()

            Picker(selection: $selectedLogLevel, label: Text("Choose Log Level")) {
                ForEach(0 ..< logLevels.count) {
                    Text(self.logLevels[$0])
                }
            }.frame(width: 400, height: 150, alignment: .center)

            Button(action: {
                switch(selectedLogLevel) {
                case 0:
                    logger.log("Default log message")
                case 1:
                    logger.info("Info log message")
                case 2:
                    logger.debug("Debug log message")
                case 3:
                    logger.error("Error log message")
                default: // 4
                    logger.fault("Fault log message")
                }
            }) {
                Text("Log with Log Level \(logLevels[selectedLogLevel])")
            }.padding()
            
            Button(action: sendLogs) {
                Text("Send logs to developers")
            }
        }
    }
}
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}

```
