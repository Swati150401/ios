# ios

struct ApiResponse: Codable {
    let data: [ApiData]
}

struct ApiData: Codable {
    let title: String
    let url: String
    let clicks: Int
    let impressions: Int
}

class ApiClient {
    func fetchData(completion: @escaping (ApiResponse?) -> Void) {
        let urlString = "https://api.inopenapp.com/api/v1/dashboardNew"
        guard let url = URL(string: urlString) else {
            completion(nil)
            return
        }

        let accessToken = "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MjU5MjcsImlhdCI6MTY3NDU1MDQ1MH0.dCkW0ox8t bjJA2GgUx2UEwNlbTZ7Rr38PVFJevYcXFI"
        var request = URLRequest(url: url)
        request.setValue(accessToken, forHTTPHeaderField: "Authorization")

        URLSession.shared.dataTask(with: request) { (data, response, error) in
            if let data = data {
                let decoder = JSONDecoder()
                do {
                    let apiResponse = try decoder.decode(ApiResponse.self, from: data)
                    completion(apiResponse)
                } catch {
                    print("Error decoding JSON: \(error)")
                    completion(nil)
                }
            } else {
                completion(nil)
            }
        }.resume()
    }
}
Display a greeting based on the local time
struct ContentView: View {
    @State private var greeting = ""

    var body: some View {
        VStack {
            Text(greeting)
                .padding()
            // ...
        }
        .onAppear {
            let dateFormatter = DateFormatter()
            dateFormatter.timeStyle = .short
            let time = dateFormatter.string(from: Date())
            let hour = Calendar.current.component(.hour, from: Date())
            if hour < 12 {

            To API request to display "Top links" and "Recent links"
            let apiBaseUrl = "https://api.inopenapp.com/api/v1/dashboardNew"
let accessToken = "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MjU5MjcsImlhdCI6MTY3NDU1MDQ1MH0.dCkW0ox8t bjJA2GgUx2UEwNlbTZ7Rr38PVFJevYcXFI"

// Modify the API URL to display Top links
let topLinksUrl = "\(apiBaseUrl)?access_token=\(accessToken)&type=top"

// Modify the API URL to display Recent links
let recentLinksUrl = "\(apiBaseUrl)?access_token=\(accessToken)&type=recent"


#
import SwiftUI

struct Link: Decodable {
    let id: Int
    let title: String
    let url: URL
    let clicks: Int
    let impressions: Int
}

struct DashboardData: Decodable {
    let links: [Link]
}

class ApiClient {
    func fetchDashboardData(url: URL, completion: @escaping (DashboardData?) -> Void) {
        URLSession.shared.dataTask(with: url) { (data, response, error) in
            if let data = data {
                let decoder = JSONDecoder()
                do {
                    let dashboardData = try decoder.decode(DashboardData.self, from: data)
                    completion(dashboardData)
                } catch {
                    print("Error decoding JSON: \(error)")
                    completion(nil)
                }
            } else {
                completion(nil)
            }
        }.resume()
    }
}

struct ContentView: View {
    @State private var topLinksData: DashboardData?
    @State private var recentLinksData: DashboardData?

    var body: some View {
        VStack {
            if topLinksData != nil && recentLinksData != nil {
                TabView {
                    LinkListView(title: "Top Links", links: topLinksData!.links)
                        .tabItem {
                            Text("Top")
                        }
                    LinkListView(title: "Recent Links", links: recentLinksData!.links)
                        .tabItem {
                            Text("Recent")
                        }
                }
            } else {
                ProgressView()
                    .onAppear {
                        let topLinksUrl = URL(string: "https://api.inopenapp.com/api/v1/dashboardNew?access_token=\(accessToken)&type=top")!
                        let recentLinksUrl = URL(string: "https://api.inopenapp.com/api/v1/dashboardNew?access_token=\(accessToken)&type=recent")!

                        ApiClient().fetchDashboardData(url: topLinksUrl) { (topLinksData) in
                            self.topLinksData = topLinksData
                        }

                        ApiClient().fetchDashboardData(url: recentLinks
