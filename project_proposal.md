Air Quality Monitoring Proposal

Project Overview
Project Title: Air Quality Monitoring
Project Type: Interactive Public Dashboard
Target Audience: Louisville, KY residents and visitors
Technology Stack: Python, Supabase Microsoft BI, Open-Meteo API

Background
Louisville Metro Area rates as the 22nd Worst City for high-ozone days and 45th for worst annual particle pollution (ALA, 2026). The American Lung Association (ALA) utilizes the Environmental Protection Agency's (EPA) national standards to produce "State of the Air" reports for cities and counties across the United States. Air pollution can cause respiratory illnesses such as increased coughing, shortness of breath, attacks, worsening COPD, and lung cancer (ALA, 2026). Additionally, air pollution has been shown to lead to premature death, increased infections, heart attacks and strokes, impaired cognitive functions, metabolic disorders, and preterm birth and low birth weight. (EPA, 2019; 2022).

Bakersfield, CA Brownfield, TX, and Eugene, OR are ranked as the worst 3 cities by having the highest / worst air quality across the US. Kahului, HI, Bozeman, MT, and Casper WY are ranked at top 3 cities by having the lowest / best AQI across the United States. These six cities are also included in the dashboard so that viewers can compare Louisville data to the extreme cities to give additional context to the data.

Open Meteo provides open access to air quality data including Particulate Matter, Ozone, UV Index, Multiple Pollen ratings, and various gasses (Nitrogen, Ammonia, Methane, Carbon Monoxide, Carbon Dioxide, Carbon, Methane, and Formaldehyde)/ Additionally, A total Air Quality Index score (AQI) is available which ranges in the United States from 0-500 with the following rating categories: 
0-50 - Good
51-100 - Moderate
101-150 - Unhealthy for Sensitive Groups
151 - 200 - Unhealthy
201 - 300 - Very Unhealthy
301 - 500 - Hazardous 

While significant historic and current health disparities exist within certain areas in the city (Hanchette et al., 2011; Ramiriz et al., 2023), knowledge of how air quality impacts one's health is important as well as having access to accurate and reliable data. This project looks to provide a comprehensive, accessible, and user-friendly dashboard for citizens and visitors in the Louisville-Metro area. Understanding the importance of air quality as well as the current AQI will help users be prepared for their day-to-day commutes, their time spent outdoors, as well as the air that is circulating through their homes. 

Problem Statement

Currently, one can access air quality index data through their phone's weather app, but this is updated, on average, once an hour to once per day depending on extreme weather conditions. AirNow.gov uses an 8–24-hour window for their measurements (EPA, 2026). However, if one is riding public transit or not at home, they may not have easy access to these sources. This dashboard will be easily accessible and available on TARC busses, through Louisville Metro services (Recycle Coach, Louisville Metro Website, and 311), as well as through an accessible Microsoft BI Dashboard.

Objectives
Primary Objectives:
- Create an interactive and accessible dashboard for citizens and visitors using Microsoft BI and Python.
- Integrate real-time Air Quality Index data from Open-Meteo for the Louisville-Metro Area.
- Incorporate interpretations and recommendations based on real-time AQI data
- Provide users with suggestions in response from AQI data (i.e., wear a mask if going outside, staying inside, if possible, etc.) 

Secondary Objectives
- Provide users with air quality education and its health impact
- Utilize community feedback to provide a usable and accessible resource 
- Ensure that the dashboard is engaging and educational for all users

Methodology / Technical Approach

Air Quality Dashboard
•	The data and API will be pulled from Open Meteo and cross-checked with AirNow 
•	The dashboard will include current conditions, 7-day forecast, interpretations, and comprehensive visuals
•	The dashboard will be created using Python, Supabase, and Power BI

ETL Workflow Diagram

<img width="1079" height="300" alt="image" src="https://github.com/user-attachments/assets/ef7dd103-c944-4d3f-b3dc-0eea5f2d4eb6" />

 
•	The data will be displayed in Microsoft Power BI

•	Data will be visualized in different Microsoft Power BI with interpretations:

o	Page 1 - AQI Overview with Interpretation using EPA guidelines and color coding and table including driving variables
o	Page 2 - Detailed Air Quality Index view
o	Page 3 - Greenhouse Gasses and Pollutants - Ozone rates per city and pollutants present in the air measured against WHO % recommendations
o	Page 4 - Map and UV Overview - Includes a heat map of AQI across the cities along with UV Index during day and night for the selected cities. 

•	The data will automatically updated when the user opens it or when a new day begins (for public facing dashboards (i.e., public transit busses)
•	The data will be cleaned by using only data with minimal missing data and data that matches Air.gov and local sources

Timeline
Week	Tasks
Week 1	Research topic, select API, draft proposal
Week 2	Build API extraction and transformation pipeline
Week 3	Complete storage layer and data cleaning
Week 4	Build dashboard, visualizations, and interpretations
Week 5	Final testing, presentation, and submission

Expected Outcomes

•	Working ETL pipeline (see workflow above)
•	Interactive dashboard with interpretations for AQI, pollen allergens, and gasses. 
•		This dashboard will include educational interpretations and recommendations
•	Automated data workflow that will create a Power BI dashboard
•	Analytical report with example data output and pages from the dashboard
•	Presentation including the data and all of the above



 
References

American Lung Association. (2026). State of the Air. https://www.lung.org/research/sota

Hanchette, C., Lee, J. H., & Aldrich, T. E. (2011). Asthma, air quality and environmental justice in Louisville, Kentucky. In Geospatial Analysis of Environmental Health (pp. 223-242). Dordrecht: Springer Netherlands.

Ramirez, J., Furmanek, S., Chandler, T. R., Wiemken, T., Peyrani, P., Arnold, F., Mattingly, W., Wilde, A., Bordon, J., Fernandez-Botran, R., Carrico, R., Cavallazzi, R., & The University of Louisville Pneumonia Study Group. (2023). Epidemiology of Pneumococcal Pneumonia in Louisville, Kentucky, and Its Estimated Burden of Disease in the United States. Microorganisms, 11(11), 2813. https://doi.org/10.3390/microorganisms11112813

United States Environmental Protection Agency. (2026). AirNow. https://www.airnow.gov/

United States Environmental Protection Agency. (2019) Integrated Science Assessment (ISA) for Particulate Matter. National Ambient Air Quality Standard (NAAQS). EPA/600/R-19/188; Sections 9.1.2.6, 6.1.2, 5.1.2.1, 5.1.2.1.1

United States Environmental Protection Agency. (2022) Integrated Science Assessment (ISA) for Particulate Matter.  National Ambient Air Quality Standard (NAAQS). EPA/600/R-22/028; Table 2-1 - Causal and likely to be causal causality determinations for short- and long-term PM2.5 exposure.




 
Appendix A
Sample Code
import { fetchWeatherApi } from "openmeteo";
const params = {
	latitude: 38.2527,
	longitude: -85.7585,
	hourly: ["pm10", "pm2_5", "dust", "formaldehyde", "carbon_monoxide", "carbon_dioxide", "nitrogen_dioxide", "sulphur_dioxide", "ozone", "alder_pollen", "aerosol_optical_depth", "uv_index", "uv_index_clear_sky", "ammonia", "methane", "ragweed_pollen", "olive_pollen", "mugwort_pollen", "grass_pollen", "birch_pollen", "us_aqi", "us_aqi_pm2_5", "us_aqi_pm10", "us_aqi_ozone"],
	current: ["us_aqi", "pm10", "ozone", "alder_pollen", "grass_pollen", "olive_pollen", "ragweed_pollen", "birch_pollen", "mugwort_pollen"],
	timezone: "America/New_York",
	forecast_days: 7,
	forecast_hours: 24,};
const url = "https://air-quality-api.open-meteo.com/v1/air-quality";
const responses = await fetchWeatherApi(url, params);

const latitude = response.latitude();
const longitude = response.longitude();
const elevation = response.elevation();
const timezone = response.timezone();
const timezoneAbbreviation = response.timezoneAbbreviation();
const utcOffsetSeconds = response.utcOffsetSeconds();

console.log(
	`\nCoordinates: ${latitude}°N ${longitude}°E`,
	`\nElevation: ${elevation}m asl`,
	`\nTimezone: ${timezone} ${timezoneAbbreviation}`,
	`\nTimezone difference to GMT+0: ${utcOffsetSeconds}s`,
);

const current = response.current()!;
const hourly = response.hourly()!;

const weatherData = {
	current: {
		time: new Date((Number(current.time()) + utcOffsetSeconds) * 1000),
		us_aqi: current.variables(0)!.value(),
		pm10: current.variables(1)!.value(),
		ozone: current.variables(2)!.value(),
		alder_pollen: current.variables(3)!.value(),
		grass_pollen: current.variables(4)!.value(),
		olive_pollen: current.variables(5)!.value(),
		ragweed_pollen: current.variables(6)!.value(),
		birch_pollen: current.variables(7)!.value(),
		mugwort_pollen: current.variables(8)!.value(),
	},
	hourly: {
		time: Array.from(
			{ length: (Number(hourly.timeEnd()) - Number(hourly.time())) / hourly.interval() }, 
			(_ , i) => new Date((Number(hourly.time()) + i * hourly.interval() + utcOffsetSeconds) * 1000)
		),
		pm10: hourly.variables(0)!.valuesArray(),
		pm2_5: hourly.variables(1)!.valuesArray(),
		dust: hourly.variables(2)!.valuesArray(),
		formaldehyde: hourly.variables(3)!.valuesArray(),
		carbon_monoxide: hourly.variables(4)!.valuesArray(),
		carbon_dioxide: hourly.variables(5)!.valuesArray(),
		nitrogen_dioxide: hourly.variables(6)!.valuesArray(),
		sulphur_dioxide: hourly.variables(7)!.valuesArray(),
		ozone: hourly.variables(8)!.valuesArray(),
		alder_pollen: hourly.variables(9)!.valuesArray(),
		aerosol_optical_depth: hourly.variables(10)!.valuesArray(),
		uv_index: hourly.variables(11)!.valuesArray(),
		uv_index_clear_sky: hourly.variables(12)!.valuesArray(),
		ammonia: hourly.variables(13)!.valuesArray(),
		methane: hourly.variables(14)!.valuesArray(),
		ragweed_pollen: hourly.variables(15)!.valuesArray(),
		olive_pollen: hourly.variables(16)!.valuesArray(),
		mugwort_pollen: hourly.variables(17)!.valuesArray(),
		grass_pollen: hourly.variables(18)!.valuesArray(),
		birch_pollen: hourly.variables(19)!.valuesArray(),
		us_aqi: hourly.variables(20)!.valuesArray(),
		us_aqi_pm2_5: hourly.variables(21)!.valuesArray(),
		us_aqi_pm10: hourly.variables(22)!.valuesArray(),
		us_aqi_ozone: hourly.variables(23)!.valuesArray(),},};


console.log(
	`\nCurrent time: ${weatherData.current.time}\n`,
	`\nCurrent us_aqi: ${weatherData.current.us_aqi}`,
	`\nCurrent pm10: ${weatherData.current.pm10}`,
	`\nCurrent ozone: ${weatherData.current.ozone}`,
	`\nCurrent alder_pollen: ${weatherData.current.alder_pollen}`,
	`\nCurrent grass_pollen: ${weatherData.current.grass_pollen}`,
	`\nCurrent olive_pollen: ${weatherData.current.olive_pollen}`,
	`\nCurrent ragweed_pollen: ${weatherData.current.ragweed_pollen}`,
	`\nCurrent birch_pollen: ${weatherData.current.birch_pollen}`,
	`\nCurrent mugwort_pollen: ${weatherData.current.mugwort_pollen}`,);
console.log("\nHourly data:\n", weatherData.hourly)
