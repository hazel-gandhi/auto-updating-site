# Auto-updating AQI Website

#### This website shows the daily AQI of approximately 250 Indian cities by pulling data from [India's Pollution Control Board](https://cpcb.nic.in/AQI_Bulletin.php).

**Link to the project:** [https://hazel-gandhi.github.io/auto-updating-site/](https://hazel-gandhi.github.io/auto-updating-site/)


## Folder Guide

### 1. `notebooks+data`
This folder is the root for all scripts. Structure is:

* All `.ipynb` files used for scraping and analysis.
* **Subfolder `/data`**: Contains all CSV files that the analysis outputs
* **Subfolder `/pdfs`**: Stores the daily PDFs downloaded and used to scrape AQI data.

## Workflow:

### 1. `step1-india-aqi-scraper.ipynb`
Extract the PDF from the pollution control website using **BeautifulSoup** and analyze it using **Natural PDF**. City data is exported to `cities.csv`.

* **1.1 Get the first PDF on the page:**
    ```python
    pdfs = soup_doc.find('ol').find('li').find('a')['href']
    ```

* **1.2 Extract the first PDF page:**
    ```python
    extracted_tables[0][0].df
    ```

* **1.3 Loop through all pages to build the AQI dataframe:**
    ```python
    dfs = []

    for page in extracted_tables.values():
        for table in page:
            # Add it to the list
            dfs.append(table.df)

    df = pd.concat(dfs, ignore_index=True)
    df.head()
    ```


### 2. `step2-geocoding.ipynb`
Import `cities.csv` to get the latitude and longitude for each city.

* **2.1 Geocoding function (Google Geocoding API):**
    ```python
    def geocode_city(city_name):
        url = "[https://maps.googleapis.com/maps/api/geocode/json](https://maps.googleapis.com/maps/api/geocode/json)"
        params = {"address": city_name, "key": API_KEY}
        
        response = requests.get(url, params=params)
        data = response.json()
        
        if data["status"] == "OK":
            location = data["results"][0]["geometry"]["location"]
            return location["lat"], location["lng"]
        else:
            print(f"Skipped {city_name}: {data.get('error_message', data['status'])}")
            return None, None
    ```

* **2.2 Generate coordinate list:**
    ```python
    results = []
    for city in unique_cities:
        lat, lng = geocode_city(city)
        results.append({"City": city, "Lat": lat, "Lng": lng})
        time.sleep(0.1)
    ```

* **2.3 Export results:** Save output to `city_coords.csv`.


### 3. `step3-data-for-chart.ipynb`
Final data cleaning and merging. This step joins the coordinates with the AQI data and exports the results to the production file.

> **Note:** The main file that updates every day is `final_aqi.csv`.


## Data Diary
A detailed breakdown of the process and challenges faced during this project is available in the [diary.md](./diary.md) file of this repository.
