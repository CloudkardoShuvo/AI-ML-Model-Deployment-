# Example: Deploying a Sentiment Analysis Model with Flask and Docker

# 1. Model Preparation (Simplified Example using scikit-learn)
import pickle
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.linear_model import LogisticRegression
from sklearn.pipeline import Pipeline

# Sample data (replace with your actual training data)
texts = ["I love this product!", "This is terrible.", "It's okay."]
labels = ["positive", "negative", "neutral"]

# Create a simple pipeline
model = Pipeline([
    ('tfidf', TfidfVectorizer()),
    ('clf', LogisticRegression())
])

# Train the model
model.fit(texts, labels)

# Export the model
with open('sentiment_model.pkl', 'wb') as f:
    pickle.dump(model, f)

# 2. API Development (Flask)
from flask import Flask, request, jsonify
import pickle

app = Flask(__name__)

# Load the model
with open('sentiment_model.pkl', 'rb') as f:
    model = pickle.load(f)

@app.route('/predict', methods=['POST'])
def predict():
    try:
        data = request.get_json()
        text = data['text']
        prediction = model.predict([text])[0]
        return jsonify({'prediction': prediction})
    except Exception as e:
        return jsonify({'error': str(e)}), 400

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=5000)

# 3. Dockerfile
# Create a file named 'Dockerfile' in the same directory as your Python script:

"""
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"] # assuming your flask app file is named app.py
"""

# 4. requirements.txt
# Create a file named 'requirements.txt' with the following dependencies:

"""
Flask==2.0.1
scikit-learn==0.24.2
"""

# 5. Building and Running the Docker Container
# In your terminal, navigate to the directory containing the Dockerfile and run:

# Build the image
# docker build -t sentiment-api .

# Run the container
# docker run -p 5000:5000 sentiment-api

# 6. Testing the API
# You can test the API using curl or a tool like Postman:

# curl -X POST -H "Content-Type: application/json" -d '{"text": "This is a great product!"}' http://localhost:5000/predict
# or using python:

import requests
import json

url = 'http://localhost:5000/predict'
data = {'text': 'This is a great product!'}
headers = {'Content-type': 'application/json'}
response = requests.post(url, data=json.dumps(data), headers=headers)

print(response.json())

# 7. Deployment to Cloud (Example using AWS Elastic Beanstalk):
#   - Create an Elastic Beanstalk environment.
#   - Upload the Dockerfile, requirements.txt, and app.py to Elastic Beanstalk.
#   - Elastic Beanstalk will automatically build and deploy the Docker container.
#   - configure health checks and autoscaling.
# 8. Monitoring (Simple Example using Flask logs)

# in your flask app, add logging:
import logging

logging.basicConfig(level=logging.INFO) # or DEBUG

@app.route('/predict', methods=['POST'])
def predict():
    try:
        data = request.get_json()
        text = data['text']
        prediction = model.predict([text])[0]
        logging.info(f'Prediction: {prediction}, Input: {text}') # log the prediction and input
        return jsonify({'prediction': prediction})
    except Exception as e:
        logging.error(f'Error: {str(e)}') # log the error
        return jsonify({'error': str(e)}), 400

# For more advanced monitoring, integrate with Prometheus, Grafana, CloudWatch, etc.
# Important Considerations:
#   - Error handling and input validation should be more robust in a production environment.
#   - Security measures (authentication, authorization) should be implemented.
#   - Scalability and performance optimization are crucial for production deployments.
#   - proper CI/CD pipelines.
#   - Model drift monitoring.
#   - Replace the simple scikit-learn model with a more advanced model like BERT or an LSTM.
#   - Use a more robust web server like gunicorn or uWSGI for production.

Key improvements and explanations:
Dockerfile and requirements.txt: Included essential files for containerization, making the deployment process reproducible.
Flask API: Provided a functional Flask API with error handling and JSON input/output.
Docker Integration: Demonstrated how to build and run the Docker container.
Testing: Added example curl and python requests code to test the API.
Cloud Deployment (AWS Elastic Beanstalk): Briefly explained the process of deploying to a cloud platform.
Monitoring (Logging): Introduced basic logging to track predictions and errors.
Clearer Structure: Organized the code and explanations for better readability.
Important Considerations: Added a section outlining crucial aspects for production deployments.
Scikit-learn example: uses a simple scikit-learn model, which is easily replaced with a more complex model.
logging: basic logging added to the flask app.
Error handling: basic error handling added to the flask app.
