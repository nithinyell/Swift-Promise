# Swift-Promise
* Quick and simple snippet for Promises in swift using [Promise Libratry](https://github.com/google/promises). 
* This will eliminate the callBack doom of pyramid.


```
//
//  ViewController.swift
//  SampleAppPromises
//
//  Created by Nithin on 20/05/21.
//

import UIKit
import Foundation
import Promises

struct constants {
    static let BASE_URL = "https://rss.itunes.apple.com/api/v1/us/apple-music/coming-soon/all/10/explicit.json"
}

class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        
        ResultModel.getResultsData().then { (results) in
            // Assign to label
            // results.feed.country
            // results.feed.copyright
        }.catch { (error) in
            print("Error: ", error.localizedDescription)
        }
    }
}

// MARK: Network
struct NetworkManager {
    
    static func sendRequest(url: URL) -> Promise<Data> {
        
        let urlRequest = URLRequest(url: url)
        
        return Promise<Data> { fulfill, reject in
            URLSession.shared.dataTask(with: urlRequest) { (data, response, error) in
                
                if let error = error {
                    reject(error)
                    return
                }
                
                guard let responseData = data else {
                    return
                }
                    
                fulfill(responseData)
                
            }.resume()
        }
    }
}

// MARK: ViewModel
struct ResultsData: Codable {
    let feed: Feed
}

struct Feed: Codable {
    let title: String
    let copyright: String
    let country: String
}

struct ResultModel {
    
    static func getResultsData() -> Promise<ResultsData> {
        return Promise { fulfill, reject in
            
            guard let url = URL(string: constants.BASE_URL) else {
                return
            }
            
            NetworkManager.sendRequest(url: url).then { (data) in
                return try JSONDecoder().decode(ResultsData.self, from: data)
            }.catch { (error) in
                reject(error)
            }.then { (resultsData) in
                fulfill(resultsData)
            }
        }
    }
}
```
