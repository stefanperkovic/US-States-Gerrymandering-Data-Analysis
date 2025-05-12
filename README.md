# US States Congressional District Gerrymandering Analysis

## Project Goal

This project aims to analyze congressional district maps for various US states to assess potential gerrymandering. It calculates several common metrics, including partisan fairness (Efficiency Gap), district shape compactness (Polsby-Popper), and the distribution of minority communities (Community Concentration Score). The results are then visualized on an interactive web-based map.

## Features

* **State-Level Analysis:** Python scripts to process voter files, CVAP data, and shapefiles for individual states.
* **Gerrymandering Metrics Calculation:**
    * **Efficiency Gap:** Measures partisan bias in election outcomes.
    * **Polsby-Popper Score:** Quantifies the geometric compactness of district shapes.
    * **Community Concentration Score:** Assesses whether minority communities are overly packed into few districts or cracked across many.
* **Interactive Map Frontend:** An HTML/JavaScript application using Leaflet.js to:
    * Display congressional districts for selected states.
    * Color-code districts by their Polsby-Popper scores.
    * Show detailed metrics for each district on click.
    * Display statewide summary metrics.
* **Data-Driven:** Uses block-level voter data (e.g., from L2), Census CVAP data, and Census TIGER/Line shapefiles.
* **Scalable:** Designed with a generic Python script template to facilitate analysis of multiple states.

## Key Metrics Calculated

1.  **Efficiency Gap:**
    * Formula: `(Total Wasted Votes for Party B - Total Wasted Votes for Party A) / Total Votes Statewide`
    * Interpretation: Positive values suggest the map favors Party A; negative values suggest it favors Party B. Values close to zero indicate more partisan fairness.
2.  **Polsby-Popper Score:**
    * Formula: `(4 * PI * Area_of_District) / (Perimeter_of_District^2)`
    * Interpretation: Scores range from 0 (least compact, very irregular) to 1 (most compact, a perfect circle).
3.  **Community Concentration Score (Sum of Squared Shares):**
    * Formula: `Sum(s_d^2)` where `s_d` is the share of statewide high-concentration tracts of a minority group located in district `d`.
    * Interpretation: Compares the score to a baseline of `1 / Number_of_Districts` (perfectly even spread). Scores closer to 1.0 indicate higher packing of the specified community into fewer districts.

## How to View the Frontend Map

1.  **Start a Local Web Server:**
    * Open your terminal.
    * Navigate to the **root of your `GerymanderingProject/` directory** (the one containing the HTML file and the `results` folder).
        ```bash
        cd /path/to/your/GerymanderingProject/
        ```
    * Run the Python HTTP server:
        ```bash
        python3 -m http.server 
        ```
        (or `python -m http.server` for Python 2/some Python 3 setups)
2.  **Open in Browser:**
    * Open your web browser and go to: `http://localhost:8000/multi_state_map_frontend_v2.html`
    * Select a state from the dropdown to view its map and analysis.

## Future Enhancements (Potential Ideas)

* Implement a national overview map showing a summary gerrymandering score for each state.
* Add more sophisticated compactness measures 
* Incorporate historical district plans for comparison.
* Allow users to upload their own proposed district plans (advanced).
* Develop more nuanced "gerrymandering tier" calculations based on multiple metrics.
* Improve mobile responsiveness of the front-end.
