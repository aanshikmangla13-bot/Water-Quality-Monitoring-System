# Water-Quality-Monitoring-System
Continuous monitoring and prediction of water quality parameters to prevent water-borne diseases and environmental damage





import pandas as pd
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
import numpy as np

class WaterQualityAnalyzer:
    def __init__(self):
        self.quality_standards = {
            'excellent': {'ph_min': 6.5, 'ph_max': 7.5, 'turbidity_max': 1, 'do_min': 8},
            'good': {'ph_min': 6.0, 'ph_max': 8.0, 'turbidity_max': 5, 'do_min': 6},
            'poor': {'ph_min': 5.0, 'ph_max': 9.0, 'turbidity_max': 10, 'do_min': 4},
            'critical': {'ph_min': 0, 'ph_max': 14, 'turbidity_max': 100, 'do_min': 0}
        }
    
    def classify_water_quality(self, ph, turbidity, dissolved_oxygen, temperature):

        score = 0
        
     
        if 6.5 <= ph <= 8.5:
            score += 3
        elif 6.0 <= ph <= 9.0:
            score += 2
        else:
            score += 0
        
    
        if turbidity <= 1:
            score += 3
        elif turbidity <= 5:
            score += 2
        elif turbidity <= 10:
            score += 1
        
        
        if dissolved_oxygen >= 8:
            score += 3
        elif dissolved_oxygen >= 6:
            score += 2
        elif dissolved_oxygen >= 4:
            score += 1
   
        if 10 <= temperature <= 25:
            score += 1
        
       
        if score >= 10:
            return "Excellent", score
        elif score >= 7:
            return "Good", score
        elif score >= 4:
            return "Fair", score
        else:
            return "Poor", score
    
    def predict_contamination_risk(self, historical_data):
 
        X = historical_data[['ph', 'turbidity', 'dissolved_oxygen', 'temperature']]
        y = historical_data['contamination_risk']
        
        X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)


        
        
        model = RandomForestClassifier(n_estimators=100)
        model.fit(X_train, y_train)
        
        accuracy = model.score(X_test, y_test)
        return model, accuracy



        implementation 

     
import pandas as pd
import numpy as np
from datetime import datetime, timedelta
from quality_analyzer import WaterQualityAnalyzer

class AquaGuardSystem:
    def __init__(self):
        self.analyzer = WaterQualityAnalyzer()
        self.water_data = pd.DataFrame(columns=[
            'timestamp', 'location', 'ph', 'turbidity', 
            'dissolved_oxygen', 'temperature', 'quality_score'
        ])
    
    def add_reading(self, location, ph, turbidity, do, temp):
        
        quality, score = self.analyzer.classify_water_quality(ph, turbidity, do, temp)
        
        new_reading = {
            'timestamp': datetime.now(),
            'location': location,
            'ph': ph,
            'turbidity': turbidity,
            'dissolved_oxygen': do,
            'temperature': temp,
            'quality_score': score,
            'quality_status': quality
        }
        
        self.water_data = pd.concat([self.water_data, pd.DataFrame([new_reading])], ignore_index=True)
        return quality, score
    
    def generate_water_report(self, location):
     
        location_data = self.water_data[self.water_data['location'] == location]
        
        if location_data.empty:
            return "No data available for this location"
        
        latest = location_data.iloc[-1]
        avg_quality = location_data['quality_score'].mean()
        
        report = f"""
        WATER QUALITY REPORT - {location}
        Generated: {datetime.now().strftime('%Y-%m-%d %H:%M')}
        
        CURRENT READING:
        - pH Level: {latest['ph']} ({'Optimal' if 6.5 <= latest['ph'] <= 8.5 else 'Suboptimal'})
        - Turbidity: {latest['turbidity']} NTU ({'Clear' if latest['turbidity'] <= 5 else 'Cloudy'})
        - Dissolved Oxygen: {latest['dissolved_oxygen']} mg/L ({'Good' if latest['dissolved_oxygen'] >= 6 else 'Low'})
        - Temperature: {latest['temperature']}°C
        - Overall Quality: {latest['quality_status']} (Score: {latest['quality_score']}/12)
        
        TREND ANALYSIS:
        - Average Quality Score: {avg_quality:.2f}/12
        - Readings Count: {len(location_data)}
        - Status: {'STABLE' if avg_quality >= 7 else 'NEEDS ATTENTION'}
        """
        
        return report

# Demo execution
if __name__ == "__main__":
    system = AquaGuardSystem()
    
    # Add sample readings
    system.add_reading("Lake Community", 7.2, 0.8, 8.5, 22)
    system.add_reading("Lake Community", 7.0, 1.2, 7.8, 23)
    system.add_reading("River Site A", 6.8, 8.5, 5.2, 25)
    
    # Generate report
    report = system.generate_water_report("Lake Community")
    print(report)
