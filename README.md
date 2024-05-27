import React, { useState } from 'react';
import axios from 'axios';

function App() {
  const [file, setFile] = useState(null);
  const [predictions, setPredictions] = useState([]);

  const handleFileChange = (e) => {
    setFile(e.target.files[0]);
  };

  const handleUpload = async () => {
    const formData = new FormData();
    formData.append('file', file);

    try {
      const response = await axios.post('http://localhost:5000/upload', formData, {
        headers: {
          'Content-Type': 'multipart/form-data',
        },
      });
      setPredictions(response.data.predictions);
    } catch (error) {
      console.error('Error uploading file:', error);
    }
  };

  return (
    <div>
      <h1>Academic Performance Prediction</h1>
      <input type="file" onChange={handleFileChange} />
      <button onClick={handleUpload}>Upload and Predict</button>
      <div>
        {predictions.length > 0 && (
          <table>
            <thead>
              <tr>
                <th>Student ID</th>
                <th>Predicted Performance</th>
              </tr>
            </thead>
            <tbody>
              {predictions.map((prediction, index) => (
                <tr key={index}>
                  <td>{prediction.studentId}</td>
                  <td>{prediction.performance}</td>
                </tr>
              ))}
            </tbody>
          </table>
        )}
      </div>
    </div>
  );
}

export default App;
const express = require('express');
const multer = require('multer');
const path = require('path');
const { predictPerformance } = require('./predict');

const app = express();
const upload = multer({ dest: 'uploads/' });

app.post('/upload', upload.single('file'), async (req, res) => {
  try {
    const filePath = path.join(__dirname, req.file.path);
    const predictions = await predictPerformance(filePath);
    res.json({ predictions });
  } catch (error) {
    console.error('Error processing file:', error);
    res.status(500).send('Error processing file');
  }
});

app.listen(5000, () => {
  console.log('Server is running on port 5000');
});
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
import joblib

def train_model():
    # Load your dataset
    data = pd.read_csv('student_data.csv')
    X = data.drop('performance', axis=1)
    y = data['performance']

    # Split data
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

    # Train model
    model = RandomForestClassifier(n_estimators=100)
    model.fit(X_train, y_train)

    # Save model
    joblib.dump(model, 'model.pkl')

def predict_performance(file_path):
    # Load model
    model = joblib.load('model.pkl')

    # Load new data
    new_data = pd.read_csv(file_path)

    # Predict performance
    predictions = model.predict(new_data)
    return [{'studentId': row['studentId'], 'performance': pred} for row, pred in zip(new_data.to_dict('records'), predictions)]
