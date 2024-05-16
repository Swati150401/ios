# ios
Before we start, here are the links you provided:

UI Template: https://iOS.openinapp.co/UITemp
API Endpoint: https://api.inopenapp.com/api/v1/dashboardNew
Access Token: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MjU5MjcsImlhdCI6MTY3NDU1MDQ1MH0.dCkW0ox8t bjJA2GgUx2UEwNlbTZ7Rr38PVFJevYcXFI
I have some questions before I begin:

What data should be displayed in the chart?
What data should be displayed in the Top links and Recent links tabs?


Assuming that the chart should display the data from the data field of the API response, and the Top links and Recent links tabs should display the data from the topLinks and recentLinks fields respectively, here's a SwiftUI implementation that should meet your requirements:


import SwiftUI
import Charts

struct ContentView: View {
    @State private var isLoading = true
    @State private var apiResponse: ApiResponse?

    var body: some View {
        NavigationView {
            Group {
                if isLoading {
                    ProgressView()
                        .onAppear(perform: loadData)
                } else {
                    ScrollView {
                        VStack(spacing: 20) {
                            greetingView
                            chartView
                            tabsView
                        }
                        .padding()
                    }
                }
            }
            .navigationTitle("Dashboard")
        }
    }

    private var greetingView: some View {
        HStack {
            if let localTime = apiResponse?.localTime {
                Text("Good \(localTime.hour >= 6 && localTime.hour < 12 ? "Morning" : localTime.hour >= 12 && localTime.hour < 18 ? "Afternoon" : "Evening"), it's \(localTime.formatted())!")
                    .font(.title3)
            }
            Spacer()
        }
    }

    private var chartView: some View {
        if let data = apiResponse?.data {
            Chart {
                ForEach(data) { point in
                    BarMark(
                        x: .value("Value", point.value),
                        y: .value("Timestamp", point.timestamp)
                    )
                    .foregroundStyle(.blue)
                }
            }
            .chartXAxis {
                AxisMarks(position: .bottom) {
                    AxisValueLabel()
                }
            }
            .chartYAxis {
                AxisMarks(position: .leading) {
                    AxisValueLabel()
                }
            }
            .frame(height: 300)
        }
    }

    private var tabsView: some View {
        TabView {
            if let topLinks = apiResponse?.topLinks {
                LinksView(links: topLinks, title: "Top Links")
            }
            if let recentLinks = apiResponse?.recentLinks {
                LinksView(links: recentLinks, title: "Recent Links")
            }
        }
        .tabViewStyle(PageTabViewStyle())
        .indexViewStyle(PageIndexViewStyle(backgroundDisplayMode: .always))
    }

    private func loadData() {
        guard let url = URL(string: "https://api.inopenapp.com/api/v1/dashboardNew") else {
            print("Invalid API URL")
            return
        }

        var request = URLRequest(url: url)
        request.setValue("Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MjU5MjcsI", forHTTPHeaderField: "Authorization")

        URLSession.shared.dataTask(with: request) { data, response, error in
            if let data = data {
                do {
                    let decodedResponse = try JSONDecoder().decode(ApiResponse.self, from: data)
                    DispatchQueue.main.async {
                        self.apiResponse = decodedResponse
                        self.isLoading = false
                    }
                } catch {
                    print("Error decoding API response: \(error)")
                }
            } else if let error = error {
                print("Network error: \(error)")
            }
        }.resume()
    }
}

Data Model:
struct ApiResponse: Decodable {
    var localTime: LocalTime
    var data: [DataPoint]
    var topLinks: [Link]
    var recentLinks: [Link]
}

struct LocalTime: Decodable {
    var hour: Int
    func formatted() -> String {
        return "\(hour):00"
    }
}

struct DataPoint: Identifiable, Decodable {
    var id: UUID
    var value: Double
    var timestamp: String
}

struct Link: Identifiable, Decodable {
    var id: UUID
    var title: String
}


Sample data : 
{
  "localTime": {
    "hour": 10
  },
  "data": [
    {"id": "1", "value": 50.0, "timestamp": "2024-05-15T10:00:00Z"},
    {"id": "2", "value": 75.0, "timestamp": "2024-05-15T11:00:00Z"},
    {"id": "3", "value": 60.0, "timestamp": "2024-05-15T12:00:00Z"}
  ],
  "topLinks": [
    {"id": "1", "title": "Top Link 1"},
    {"id": "2", "title": "Top Link 2"}
  ],
  "recentLinks": [
    {"id": "3", "title": "Recent Link 1"},
    {"id": "4", "title": "Recent Link 2"}
  ]
}


Link View:
struct LinksView: View {
    var links: [Link]
    var title: String

    var body: some View {
        VStack {
            Text(title)
                .font(.title2)
            ForEach(links) { link in
                Text(link.title)
            }
        }
    }
}
