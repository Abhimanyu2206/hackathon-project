import json
import cohere


def load_dataset(file_path):
    with open(file_path, "r", encoding="utf-8") as file:
        return json.load(file)


def query_cohere(question, context):
    co = cohere.Client('7sFYRdN8AKKbOq4e5YmWPqbAv5uwCDPRv7NRWtjJ')  


    prompt = f"Context: {context}\n\nQuestion: {question}\nAnswer:"

    try:
        response = co.generate(
            model='xlarge', 
            prompt=prompt,
            max_tokens=50,  
            temperature=0.7,  
            stop_sequences=["\n"]  
        )
        answer = response.generations[0].text.strip()
        return answer

    except Exception as e:
        return f"API Error: {e}"


def classify_question(question):
    level_1_keywords = ["what", "when", "where", "who"]  
    level_2_keywords = ["describe", "explain", "list", "tell me about"]  
    level_3_keywords = ["how", "why", "analyze", "suggest", "impact"]  
    question_lower = question.lower()

    if any(word in question_lower for word in level_1_keywords):
        return 1
    elif any(word in question_lower for word in level_2_keywords):
        return 2
    elif any(word in question_lower for word in level_3_keywords):
        return 3
    else:
        return 0  


def process_question(question, dataset):
    
    if isinstance(dataset, list) and len(dataset) > 0:
        context = dataset[0].get("context", "") 
    else:
        context = dataset.get("context", "")  

    level = classify_question(question)
    print(f"\nDetected Level: {level}")

    
    answer = query_cohere(question, context)

    print(f"\nQuestion: {question}")
    print(f"Answer: {answer}\n")

    return {"level": level, "question": question, "answer": answer}


if __name__ == "__main__":
    print("🚀 Amrita University Q&A System 🚀")
    print("Type your question or type 'exit' to quit.")

   
    dataset = load_dataset("Campus_Pal.json")

    while True:
        user_input = input("\nAsk a question: ").strip()
        
        if user_input.lower() == "exit":
            print("Goodbye! 👋")
            break
        
        
        process_question(user_input, dataset)
