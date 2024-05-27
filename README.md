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
