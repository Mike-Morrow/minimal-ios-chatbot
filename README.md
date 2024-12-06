## Minimal Chatbot frontend for iOS 
This app is a basic wrapper for using an OpenAI key. 

If you're creating this from scratch in xcode, you can copy the below code into ContentView, enter in your OpenAI key, and you're off to the races. 

```
import SwiftUI

struct ContentView: View {
    @State private var userInput: String = ""
    @State private var chatMessages: [String] = ["Welcome to the chatbot!"]
    @State private var isTyping: Bool = false

    var body: some View {
        VStack {
            HStack {
                Text("Chatbot")
                    .font(.title)
                    .bold()
                Spacer()
                Button(action: startNewChat) {
                    Text("New Chat")
                        .padding(.horizontal)
                        .padding(.vertical, 8)
                        .background(Color.blue)
                        .foregroundColor(.white)
                        .cornerRadius(10)
                }
            }
            .padding()

            ScrollView {
                VStack(alignment: .leading) {
                    ForEach(chatMessages, id: \.self) { message in
                        Text(message)
                            .padding()
                            .background(Color.gray.opacity(0.2))
                            .cornerRadius(10)
                            .padding(.vertical, 4)
                    }

                    if isTyping {
                        HStack {
                            Text("Bot is typing...")
                                .italic()
                                .foregroundColor(.gray)
                            Spacer()
                        }
                        .padding(.vertical, 4)
                    }
                }
            }
            .padding()

            HStack {
                TextField("Type your message...", text: $userInput)
                    .textFieldStyle(RoundedBorderTextFieldStyle())
                    .padding()

                Button(action: sendMessage) {
                    Text("Send")
                        .padding()
                        .background(Color.blue)
                        .foregroundColor(.white)
                        .cornerRadius(10)
                }
            }
            .padding()
        }
    }

    func startNewChat() {
        chatMessages = ["Welcome to the chatbot!"]
        userInput = ""
    }

    func sendMessage() {
        guard !userInput.isEmpty else { return }
        let message = userInput
        chatMessages.append("You: \(message)")
        userInput = ""

        isTyping = true

        fetchOpenAIResponse(for: message) { response in
            DispatchQueue.main.async {
                isTyping = false
                if let response = response {
                    chatMessages.append("Bot: \(response)")
                } else {
                    chatMessages.append("Bot: I couldn't process that request.")
                }
            }
        }
    }

    func fetchOpenAIResponse(for message: String, completion: @escaping (String?) -> Void) {
        guard let url = URL(string: "https://api.openai.com/v1/chat/completions") else {
            completion(nil)
            return
        }

        let apiKey = "your_openai_key"
        let requestBody: [String: Any] = [
            "model": "gpt-3.5-turbo",
            "messages": [
                ["role": "system", "content": "You are a helpful assistant."],
                ["role": "user", "content": message]
            ]
        ]

        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.addValue("Bearer \(apiKey)", forHTTPHeaderField: "Authorization")
        request.addValue("application/json", forHTTPHeaderField: "Content-Type")
        request.httpBody = try? JSONSerialization.data(withJSONObject: requestBody)

        URLSession.shared.dataTask(with: request) { data, _, error in
            guard let data = data, error == nil else {
                completion("Failed to fetch response")
                return
            }

            let responseJSON = try? JSONSerialization.jsonObject(with: data) as? [String: Any]
            let choices = responseJSON?["choices"] as? [[String: Any]]
            let firstChoice = choices?.first?["message"] as? [String: Any]
            let content = firstChoice?["content"] as? String
            completion(content)
        }.resume()
    }
}
```
