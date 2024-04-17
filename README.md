

# Hospitality Domain-Dashboard

 

## Problem Statement

This dashboard helps the Hotels to understand their customers better. It helps the hotels know if their customers are satisfied with their services. Through different ratings, they get to know their improvement area, & thus they can improve their services by identifying these area. It also lets them know the average rating & Revenue by category, thus since by using this dashboard they have identified this problem, they can further work on factors responsible for these unwanted delays.


### Steps followed 

- Step 1 : Load data into Power BI Desktop, dataset is a csv file.
- Step 2 : Open power query editor & in view tab under Data preview section, check "column distribution", "column quality" & "column profile" options.
- Step 3 : Also since by default, profile will be opened only for 1000 rows so you need to select "column profiling based on entire dataset".
- Step 4 : This file contains all the meta information regarding the columns described in the CSV files. we have provided 5 CSV files:
1. dim_date
2. dim_hotels
3. dim_rooms
4. fact_aggregated_bookings
5. fact_bookings


Column Description for dim_date:
1. date: This column represents the dates present in May, June and July.
2. mmm yy: This column represents the date in the format of mmm yy (monthname year).
3. week no: This column represents the unique week number for that particular date.
4. day_type: This column represents whether the given day is Weekend or Weekeday.



Column Description for dim_hotels:
1. property_id: This column represents the Unique ID for each of the hotels.
2. property_name: This column represents the name of each hotel.
3. category: This column determines which class[Luxury, Business] a particular hotel/property belongs to. 
4. city: This column represents where the particular hotel/property resides in.



Column Description for dim_rooms:
1. room_id: This column represents the type of room[RT1, RT2, RT3, RT4] in a hotel.
2. room_class: This column represents to which class[Standard, Elite, Premium, Presidential] particular room type belongs.


Column Description for fact_aggregated_bookings:
1. property_id: This column represents the Unique ID for each of the hotels.
2. check_in_date: This column represents all the check_in_dates of the customers.
3. room_category: This column represents the type of room[RT1, RT2, RT3, RT4] in a hotel.
4. successful_bookings: This column represents all the successful room bookings that happen for a particular room type in that hotel on that particular date.
5. capacity: This column represents the maximum count of rooms available for a particular room type in that hotel on that particular date.



Column Description for fact_bookings:
1. booking_id: This column represents the Unique Booking ID for each customer when they booked their rooms.
2. property_id: This column represents the Unique ID for each of the hotels
3. booking_date: This column represents the date on which the customer booked their rooms.
4. check_in_date: This column represents the date on which the customer check-in(entered) at the hotel.
5. check_out_date: This column represents the date on which the customer check-out(left) of the hotel.
6. no_guests: This column represents the number of guests who stayed in a particular room in that hotel.
7. room_category: This column represents the type of room[RT1, RT2, RT3, RT4] in a hotel.
8. booking_platform: This column represents in which way the customer booked his room.
9. ratings_given: This column represents the ratings given by the customer for hotel services.
10. booking_status: This column represents whether the customer cancelled his booking[Cancelled], successfully stayed in the hotel[Checked Out] or booked his room but not stayed in the hotel[No show].
11. revenue_generated: This column represents the amount of money generated by the hotel from a particular customer.
12. revenue_realized: This column represents the final amount of money that goes to the hotel based on booking status. If the booking status is cancelled, then 40% of the revenue generated is deducted and the remaining is refunded to the customer. If the booking status is Checked Out/No show, then full revenue generated will goes to hotels.


        
- Step 5 : New measure was calculated and following DAX expression was written =
	      
1	Revenue	To get the total revenue_realized	Revenue = SUM(fact_bookings[revenue_realized])	fact_bookings

2	Total Bookings	To get the total number of bookings happened	Total Bookings = COUNT(fact_bookings[booking_id])	fact_bookings

3	Total Capacity	To get the total capacity of rooms present in hotels	Total Capacity = SUM(fact_aggregated_bookings[capacity])	fact_aggregated_bookings

4	Total Succesful Bookings	To get the total succesful bookings happened for all hotels	Total Succesful Bookings = SUM(fact_aggregated_bookings[successful_bookings])	fact_aggregated_bookings

5	Occupancy %	"Occupancy means total successful bookings happened to the 
total rooms available(capacity)"	Occupancy % = DIVIDE([Total Succesful Bookings],[Total Capacity],0)	fact_aggregated_bookings

6	Average Rating	Get the average ratings given by the customers	Average Rating = AVERAGE(fact_bookings[ratings_given])	fact_bookings

7	No of days	"To get the total number of days present in the data.
In our case, we have data from May to July. So 92 days.

"	No of days = DATEDIFF(MIN(dim_date[date]),MAX(dim_date[date]),DAY) +1	dim_date

8	Total cancelled bookings	To get the"Cancelled" bookings out of all Total bookings happened	Total cancelled bookings = CALCULATE([Total Bookings],fact_bookings[booking_status]="Cancelled")	fact_bookings

9	Cancellation %	"calculating the cancellaton percentage.

"	Cancellation % = DIVIDE([Total cancelled bookings],[Total Bookings])	fact_bookings

10	Total Checked Out	To get the successful 'Checked out' bookings out of all Total bookings happened	Total Checked Out = CALCULATE([Total Bookings],fact_bookings[booking_status]="Checked Out")	fact_bookings

11	Total no show bookings	"To get the""No Show"" bookings out of all Total bookings happened 

(""No show"" means those customers who neither cancelled nor attend to their booked rooms)"	Total no show bookings = CALCULATE([Total Bookings],fact_bookings[booking_status]="No Show")	fact_bookings

12	No Show rate %	calculating the no show percentage.	No Show rate % = DIVIDE([Total no show bookings],[Total Bookings])	fact_bookings

13	Booking % by Platform	"To show the percentage contribution of each booking platform for bookings in hotels.

We have booking platforms like makeyourtrip, logtrip, tripster etc)"	"Booking % by Platform = DIVIDE([Total Bookings],
 CALCULATE([Total Bookings], 
 ALL(fact_bookings[booking_platform])
  ))*100"	fact_bookings

14	Booking % by Room class	"To show the percentage contribution of each room class
over total rooms booked.

We have room classes like Standard, Elite, Premium, Presidential."	"Booking % by Room class = DIVIDE([Total Bookings],
 CALCULATE([Total Bookings], 
 ALL(dim_rooms[room_class])
  ))*100"	fact_bookings, dim_rooms

15	ADR 	"Calculate the ADR(Average Daily rate)

It is the ratio of revenue to the total rooms booked/sold. 
It is the measure of the average paid for rooms sold in a given time period"	ADR = DIVIDE( [Revenue], [Total Bookings],0)	fact_bookings

16	Realisation %	"calculate  the realisation percentage.

It is nothing but the succesful ""checked out"" percentage over all bookings happened.

"	Realisation % = 1- ([Cancellation %]+[No Show rate %])	fact_bookings

17	RevPAR	"Calculate the RevPAR(Revenue Per Available Room)

RevPAR represents the revenue generated per available room, whether or not they are occupied. RevPAR helps hotels measure their revenue generating performance to accurately price rooms. RevPAR can help hotels measure themselves against other properties or brands."	RevPAR = DIVIDE([Revenue],[Total Capacity])	fact_bookings, fact_agg_bookings

18	DBRN	"calculate DBRN(Daily Booked Room Nights)

This metrics tells on average how many rooms are booked for a day considering a time period

"	DBRN = DIVIDE([Total Bookings], [No of days])	fact_bookings, dim_date

19	DSRN 	"calculate DSRN(Daily Sellable Room Nights)

This metrics tells on average how many rooms are ready to sell for a day considering a time period

"	DSRN = DIVIDE([Total Capacity], [No of days])	fact_agg_bookings, dim_date

20	DURN	"calculate DURN(Daily Utilized Room Nights)

This metric tells on average how many rooms are succesfully utilized by customers for a day considering a time period
"	DURN = DIVIDE([Total Checked Out],[No of days])	fact_bookings, dim_date

21	Revenue WoW change %	"To get the revenue change percentage week over week.

Here, 
revcw  for current week
revpw for previous week


"	"Revenue WoW change % = 
Var selv = IF(HASONEFILTER(dim_date[wn]),SELECTEDVALUE(dim_date[wn]),MAX(dim_date[wn]))
var revcw = CALCULATE([Revenue],dim_date[wn]= selv)
var revpw =  CALCULATE([Revenue],FILTER(ALL(dim_date),dim_date[wn]= selv-1))

return


DIVIDE(revcw,revpw,0)-1"	dim_date

22	Occupancy WoW change %	"To get the occupancy change percentage week over week.

Here, 
revcw  for current week
revpw for previous week"	"Occupancy WoW change % = 
Var selv = IF(HASONEFILTER(dim_date[wn]),SELECTEDVALUE(dim_date[wn]),MAX(dim_date[wn]))
var revcw = CALCULATE([Occupancy %],dim_date[wn]= selv)
var revpw =  CALCULATE([Occupancy %],FILTER(ALL(dim_date),dim_date[wn]= selv-1))

return


DIVIDE(revcw,revpw,0)-1"	dim_date

23	ADR WoW change %	"To get the ADR(Average Daily rate) change percentage week over week.

Here, 
revcw  for current week
revpw for previous week"	"ADR WoW change % = 
Var selv = IF(HASONEFILTER(dim_date[wn]),SELECTEDVALUE(dim_date[wn]),MAX(dim_date[wn]))
var revcw = CALCULATE([ADR],dim_date[wn]= selv)
var revpw =  CALCULATE([ADR],FILTER(ALL(dim_date),dim_date[wn]= selv-1))

return


DIVIDE(revcw,revpw,0)-1"	dim_date

24	Revpar WoW change %	"To get the RevPar(Revenue Per Available Room) change percentage week over week.

Here, 
revcw  for current week
revpw for previous week"	"Revpar WoW change % = 
Var selv = IF(HASONEFILTER(dim_date[wn]),SELECTEDVALUE(dim_date[wn]),MAX(dim_date[wn]))
var revcw = CALCULATE([RevPAR],dim_date[wn]= selv)
var revpw =  CALCULATE([RevPAR],FILTER(ALL(dim_date),dim_date[wn]= selv-1))

return


DIVIDE(revcw,revpw,0)-1"	dim_date

25	Realisation WoW change %	"To get the Realisation change percentage week over week.

Here, 
revcw  for current week
revpw for previous week"	"Realisation WoW change % = 
Var selv = IF(HASONEFILTER(dim_date[wn]),SELECTEDVALUE(dim_date[wn]),MAX(dim_date[wn]))
var revcw = CALCULATE([Realisation %],dim_date[wn]= selv)
var revpw =  CALCULATE([Realisation %],FILTER(ALL(dim_date),dim_date[wn]= selv-1))

return

DIVIDE(revcw,revpw,0)-1"	dim_date

26	DSRN WoW change %	"To get the DSRN(Daily Sellable Room Nights) change percentage week over week.

Here, 
revcw  for current week
revpw for previous week"	"DSRN WoW change % = 
Var selv = IF(HASONEFILTER(dim_date[wn]),SELECTEDVALUE(dim_date[wn]),MAX(dim_date[wn]))
var revcw = CALCULATE([DSRN],dim_date[wn]= selv)
var revpw =  CALCULATE([DSRN],FILTER(ALL(dim_date),dim_date[wn]= selv-1))

return


DIVIDE(revcw,revpw,0)-1"	dim_date
				


# Snapshot of Dashboard (Power BI Service)

![dashboard_snapo](https://github.com/pankajm333/Revenue-Insights-in-Hospitality-Domain/assets/111896613/550d6ee4-99eb-4f49-9b13-05a21b7446e6)



 
 # Report Snapshot (Power BI DESKTOP)

 
![Dashboard_upload](https://github.com/pankajm333/Revenue-Insights-in-Hospitality-Domain/assets/111896613/138b9d7a-bd1f-4313-a787-2140486a4a8c)



Thankyou.      
