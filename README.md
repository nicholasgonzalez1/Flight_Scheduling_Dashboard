# Flight Scheduling Dashboard
<!-- TABLE OF CONTENTS -->
<details open="open">
  <summary>Table of Contents</summary>
  <ol>
    <li><a href="#about-the-project">About the Project</a></li>
    <li><a href="#built-with">Built With</a></li>
    <li><a href="#getting-started">Getting Started</a></li>
    <li><a href="#statistical-model-for-delay-prediction">Statistical Model for Delay Prediction</a></li>
    <li><a href="#heuristic-for-minimizing-gate-utilization">Heuristic for Minimizing Gate Utilization</a></li>
    <li>
      <a href="#user-interface-walkthrough">User Interface Walkthrough</a>
      <ul>
        <li><a href="#search-criteria-and-filters">Search Criteria and Filters</a></li>
        <li><a href="#view-more-flight-information">View More Flight Information</a></li>
        <li><a href="#reschedule-a-flight">Reschedule a Flight</a></li>
        <li><a href="#cancel-a-flight">Cancel a Flight</a></li>
      </ul>
    </li>
    <li><a href="#acknowledgements">Acknowledgements</a></li>
  </ol>
</details>

<!-- ABOUT THE PROJECT -->
## About the Project

The goal of this project was to improve flight scheduling for Orlando International Airport (MCO). Our team was tasked with using historical flight records from January 1st and 2nd of 2015 to predict the flight status for the 'then' incoming flights of January 3rd through 6th of 2015.<br><br>After using a  statistical model to determine a flight's delay status, we implemented a simple heuristic for assigning them to a respective airport gate that also ensured to minimize the overall number of gates being utilized. An interactive gantt chart styled dashboard was then designed to display all appropriately scheduled incoming flights for MCO.
<br><br>
<kbd>
<img src="https://github.com/nicholasgonzalez1/Flight_Scheduling_Dashboard/blob/main/images/gui_screen.JPG?raw=true" width="800">
</kbd><br>

<!-- BUILT WITH -->
## Built With
Due to the subject material of the course (Spreadsheet Decision Support Systems), our team was limited to only using the following software and languages.
* Excel
* VBA

<!-- GETTING STARTED -->
## Getting Started

The final version of the user interface can be downloaded off [the project's repository](https://github.com/nicholasgonzalez1/Flight_Scheduling_Dashboard/blob/main/Flight%20Scheduling.xlsm). The file must be downloaded as a macro-enabled worksheet. 
<br><br>
**Note:** this project was implemented and tested on a Windows PC. It is possible that unforeseen technical issues can arise if the file is downloaded on MacOS.

<!-- USAGE EXAMPLES -->
## Statistical Model for Delay Prediction
The model used in this project is based off the Naive Bayes Classifier. It was used two separate times: first, to predict whether a flight would be delayed and second, to determine its delay amount.<br><br>Given a set of attributes for a flight (e.g. airport, airline, departure time of day), we needed to determine the **probability that it would fall under a specific delay category.** This probability was calculated using the following formula where
- C<sub>k</sub>, the delay category *k*
- X, the set of flight attributes of length *n*
- x<sub>i</sub>, a specific attribute in that set.

<kbd>
<img src="https://github.com/nicholasgonzalez1/Flight_Scheduling_Dashboard/blob/main/images/naive_bayes.JPG?raw=true" width="400">
</kbd><br><br>
    
Due to a limited data set, we estimated the conditional probabilities using the formula below where
- n hat, the number of historial flights in which C=C<sub>k</sub> and X=x
- n, the number of historical flights where C=C<sub>k</sub>
- p, equal to 1 / number of possible values for attribute *i*
- m, an equivalent sample size.

<kbd>
<img src="https://github.com/nicholasgonzalez1/Flight_Scheduling_Dashboard/blob/main/images/conditional_density_estimation.JPG?raw=true" width="275">
</kbd><br><br>

Pivot tables were created to summarize the historical flight dataset (flights from January 1st and 2nd) and determine the n hat, n, and p values.

<kbd>
<img src="https://github.com/nicholasgonzalez1/Flight_Scheduling_Dashboard/blob/main/images/pivot_tables.JPG?raw=true" width="600">
</kbd><br><br>

These values were then referenced to calculate the conditional probabilities for each incoming flight (flights from January 3rd through 6th). As you can see in column G of the image below, the conditional probabilities reads as follows, "Given that a flight is delayed, what is the probability that the flight operates under this airline?". In the case of row 9, "this airline" would be WN, Southwest Airlines. 

<kbd>
<img src="https://github.com/nicholasgonzalez1/Flight_Scheduling_Dashboard/blob/main/images/jan_3_6_data.JPG?raw=true" width="600">
</kbd><br><br>

Once all P(C<sub>k</sub> | X) were calculated for all delay categories *k*, the largest P(C<sub>k</sub> | X) value determined the predicted class, as seen in the image below.

<kbd>
<img src="https://github.com/nicholasgonzalez1/Flight_Scheduling_Dashboard/blob/main/images/decision_rule.JPG?raw=true" width="315">
</kbd><br><br>

As mentioned earlier, this statistical model was used twice. The first time around considered all flight records, and was trying to determine C=Delayed or C=Not Delayed. The second time this model was used, only flights that had been previously been determined as delayed were considered. However, this time we were trying to calculate what delay bin a flight would fall under: C=(5-15 min delay), C=(15-45 min delay), or C=(>45 min delay).
    
## Heuristic for Minimizing Gate Utilization

For each terminal, a collection of flights are assigned to gates depending on their respective gate arrival and departure times. The implemented heuristic focuses on minimizing the amount of gates utilized within each terminal. As a result, this leads to fewer flight attendants needed to manage the terminal and therefore, cheaper operational costs for MCO. The `AssignGates()` function is located here: Excel Developer Tab > Visual Basic > VBAProject > Modules > Optimizer.

### Heuristic Steps

1. The heuristic loops through each flight (aka each ???flight i???) passed in from the flight collection object, `myFlights`, and looks to assign each one to a gate within the `myGates` collection object which is currently empty at the start of the heuristic.<br><br>
2. Within each iteration, there???s another nested loop that compares gate arrival/departure times for ???flight i??? to that of the last flight assigned to each existing gate (???flight j???).<br><br>
3. If the gate times for ???flight i??? do not overlap with the times for ???flight j???, then ???flight i??? is assigned to that gate and the outer loop continues (a new ???flight i???).<br><br>
4. If there is a time conflict between ???flight i??? and ???flight j???, we consider the last flight assigned for the next existing gate (a new ???flight j???).<br><br>
5. If flight i conflicts with all flight j???s, a new gate is created and flight i is added to that gate.<br><br>
6. Each time the next iteration of the outer for loop begins, all gates in the `myGates` object are sorted so that the nested loop considers first the gates with the least amount of flights assigned to them.

```vb
Public Function AssignGates(ByVal myFlights As Collection, ByVal Airside As Integer) As Collection

    'myFlights is a collection of Flight objects. Each flight object contains a gate arrival time and gate departure time.
    
    'variable declarations
    Dim i As Integer
    Dim j As Integer
    Dim num As Integer
    Dim arrival_j As Double
    Dim departure_j As Double
    Dim arrival_i As Double
    Dim departure_i As Double
    Dim val As Integer
    i = 0
    
    'these if statements determine gate numbers as set by MCO's actual gate numbering system
    'for example, in Airside 2 of MCO, gates begin at number 100 (at the time of this project's completion)
    If (Airside = 1) Then
        num = 1
    ElseIf (Airside = 2) Then
        num = 100
    ElseIf (Airside = 3) Then
        num = 30
    Else
        num = 70
    End If
    
    'myGates is a collection of gates, where each gate contains a collection of flight objects assigned to that gate.
    'myGates is currently empty because the algorithm has not started yet
    Dim myGates As New Collection
    Dim aGate As Gate
    Dim gateFound As Boolean
    
    'loops through all Flight objects in myFlights
    For i = 1 To myFlights.Count 
    
        'Purpose of this initial if statement:
        'If this is the first flight being evaluated, then there are no gates in myGates yet
        'Therefore, immediately create a new gate and assign that flight to it
        If (i = 1) Then
            Set aGate = New Gate
            aGate.SetGateNumber (num)
            myFlights.Item(i).SetGate (num)
            aGate.AddFlight myFlights.Item(i)
            myGates.Add aGate
            num = num + 1
        Else
            gateFound = False
            arrival_i = myFlights.Item(i).GetArrival() 'for flight being evaluated, we grab arrival/departure times, these will be used
            departure_i = myFlights.Item(i).GetDeparture() 'to determine potential time conflicts for flights already scheduled in a gate
            
            'Some flights might arrive at a gate before 23:59 and then leave the next day (after 0:00)
            'for sake of the algorithm, if such case occurs, we change departure_i to 23:59.
            'Note that the departure_i is just a variable and actual flight data is not altered
            If (departure_i < arrival_i) Then
                departure_i = 0.999305555555556   'represents 23:59 in decimal notation
            End If
            
            'This next statement sorts all gates in myGates so that the gates with least amount of flights assigned
            'are considered first. This prevents some gates from having more flights assigned than other gates
            Set myGates = sortGatesByFlightNumber(myGates)
            
            'loops through all gate object in myGates to consider first and best gate avaialable
            For j = 1 To myGates.Count
                
                arrival_j = myGates.Item(j).LastFlight().GetArrival() 'grabs arrival/departure times of the last flight in gate j.
                departure_j = myGates.Item(j).LastFlight().GetDeparture() 'gate j is the gate that is currently being considered
                
                If (departure_j < arrival_j) Then
                    arrival_j = 0
                End If
                
                'this if statement ensures that times do not conflict for the flights being evaluated
                '-1.51427469135803E-03 represents the minimum time amount that must occur in between flights
                If ((departure_j - arrival_i) * (departure_i - arrival_j) < -1.51427469135803E-03) Then
                
                    'if we enter this if statement, that means we have found the first and best gate available.
                    'We mark gateFound as true and stop all other gate evaluation
                    myFlights.Item(i).SetGate (myGates.Item(j).GetGateNumber())
                    myGates.Item(j).AddFlight myFlights.Item(i)
                    gateFound = True
                    Exit For
                    
                End If
                
            Next
            
            'If we reach this point and gateFound still equals False, 
            'then we have not found an available gate for the flight and must create a new gate
            If (Not gateFound) Then
                Set aGate = New Gate
                aGate.SetGateNumber (num)
                myFlights.Item(i).SetGate (num)
                aGate.AddFlight myFlights.Item(i)
                myGates.Add aGate
                num = num + 1
            End If
            
        End If
        
        'The code above is for assigning a gate to one Flight object. 
        'The Next iteration represents the next Flight object to be assessed.
        
    Next
    
    Set AssignGates = myGates 'this is the return statement

End Function
```

## User Interface Walkthrough

Screenshots are provided below showcasing how to use the primary features of the main user interface. The code behind the interface's functionality can be found here: Excel Developer Tab > Visual Basic > VBAProject > Microsoft Excel Objects > Sheet1 (GUI).

### Search Criteria and Filters
After entering the main screen, the user can toggle the search criteria (date, time window, airside/terminal) to view currently scheduled flights in the system. For a more specific search, use the provided search filters (airline, delay). **Important:** to update the dashboard with the new search criteria/filters, make sure to press the 'Search Flights' button.
<br><br>
<kbd>
<img src="https://github.com/nicholasgonzalez1/Flight_Scheduling_Dashboard/blob/main/images/walkthrough_1.png?raw=true" width="800">
</kbd>
### View More Flight Information
To view more details about a specific flight, double click any cell with a scheduled flight. The GUI will transition to another tab displaying more information about that flight.
<br><br>
<kbd>
<img src="https://github.com/nicholasgonzalez1/Flight_Scheduling_Dashboard/blob/main/images/flight_info_screen.JPG?raw=true" width="800">
</kbd>
### Reschedule a Flight
To reschedule a flight, select one of its scheduled cells and then press 'Reschedule Flight'. A menu will appear with two input fields. If the user would like to reschedule the flight to a time slot on another date and/or time window, they would select the inputs from the combo boxes and then press 'Accept New Location'. However, if the user wants to reschedule the flight to a time on the current date and time window, they would press 'Use Current Location'.
<br><br>
<kbd>
<img src="https://github.com/nicholasgonzalez1/Flight_Scheduling_Dashboard/blob/main/images/walkthrough_3.png?raw=true" width="800">
</kbd>

After selecting either 'Use Current Location' or 'Accept New Location', the user will be prompted to highlight the flight's new location. After pressing 'OK', the system will verify that the flight's new arrival time is valid and that it does not overlap with any other scheduled flights.
<br><br>
<kbd>
<img src="https://github.com/nicholasgonzalez1/Flight_Scheduling_Dashboard/blob/main/images/walkthrough_4.png?raw=true" width="800">
</kbd>
### Cancel a Flight
To cancel a flight, select one of its scheduled cells and then press 'Cancel Flight'. A menu will appear prompting the user to confirm that they do want to cancel the flight. If 'Yes' is selected, the flight will no longer appear on the GUI nor will it exist in the system.
<br><br>
<kbd>
<img src="https://github.com/nicholasgonzalez1/Flight_Scheduling_Dashboard/blob/main/images/walkthrough_6.png?raw=true" width="800">
</kbd>

<!-- ACKNOWLEDGEMENTS -->
## Acknowledgements
Thank you to my other team members:
- [Anthony Joanes](https://www.linkedin.com/in/ajoanes)
- [Ivan Trpkovski](https://www.linkedin.com/in/ivan-trp2345)
- [Samantha Pertsas](https://www.linkedin.com/in/samantha-pertsas-cssgb-72239b136)
- [Tej Patel](https://www.linkedin.com/in/tejmpatel/)
- [Wade Niles](https://www.linkedin.com/in/wade-niles-4091381aa)
- Erin Dempsey
