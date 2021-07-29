# Flight Scheduling Dashboard

<!-- ABOUT THE PROJECT -->
## About The Project

The goal of this project was to improve flight scheduling for Orlando International Airport (MCO). Our team was tasked with using historical flight records from January 1st and 2nd of 2015 to predict the flight status for the 'then' incoming flights of January 3rd through 6th of 2015. After using a  statistical model to determine a flight's delay status, we implemented a simple heuristic for assigning them to a respective airport gate that also ensured to minimize the overall number of gates being utilized. An interactive gantt chart styled dashboard was then designed to display all appropriately scheduled incoming flights for MCO.
<br />
<br />
<img src="https://github.com/nicholasgonzalez1/Flight_Scheduling_Dashboard/blob/main/images/gui_screen.JPG?raw=true" width="800">

### Built With
* Excel
* VBA

<!-- GETTING STARTED -->
## Getting Started

You can download the final version of the user interface off [the project's repository](https://github.com/nicholasgonzalez1/Flight_Scheduling_Dashboard/blob/main/Flight%20Scheduling.xlsm). The file must be downloaded as a macro-enabled worksheet.

<!-- USAGE EXAMPLES -->

## Statistical Model for Delay Prediction
The model used in this project is based off the Naive Bayes Classifier. It was used two separate times: first, to predict whether a flight would be delayed and second, to determine its delay amount.<br><br>Given a set of attributes for a flight (e.g. airport, airline, departure time of day), we needed to determine the **probability that it would fall under a specific delay category.** This probability was calculated using the following formula where C<sub>k</sub> represents the delay category *k*; X, the set of flight attributes of length *n*; and x<sub>i</sub>, a specific attribute in that set.

<img src="https://github.com/nicholasgonzalez1/Flight_Scheduling_Dashboard/blob/main/images/naive_bayes.JPG?raw=true" width="400">

Due to a limited data set, we estimated the conditional densities using the formula below where \hat{n} represents the number of historial flights in which C=C<sub>k</sub> and X=x; n, the number of historical flights where C=C<sub>k</sub>; p, equal to 1 / number of possible values for attribute *i*; and m, an equivalent sample size.

<img src="https://github.com/nicholasgonzalez1/Flight_Scheduling_Dashboard/blob/main/images/conditional_density_estimation.JPG?raw=true" width="275">

## Heuristic for Minimizing Gate Utilization

## User Interface Walkthrough

Use this space to show useful examples of how a project can be used. Additional screenshots, code examples and demos work well in this space. You may also link to more resources.

_For more examples, please refer to the [Documentation](https://example.com)_

<!-- ACKNOWLEDGEMENTS -->
## Acknowledgements

* []()
* []()
* []()
