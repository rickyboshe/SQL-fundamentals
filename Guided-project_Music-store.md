Guided project: Music SQLite
================
Fredrick Boshe
19/04/2021

``` r
conn<-dbConnect(SQLite(), "chinook.db")
tables <- dbListTables(conn)
tables
```

    ##  [1] "album"          "artist"         "customer"       "employee"      
    ##  [5] "genre"          "invoice"        "invoice_line"   "media_type"    
    ##  [9] "playlist"       "playlist_track" "track"

``` sql
--Preview the tables in the database
SELECT
    name,
    type
FROM sqlite_master
WHERE type IN ("table","view");

--Most sold genres
SELECT g.name as genre_name, il.quantity as quantity_sold
FROM genre as g
LEFT JOIN track as t ON t.genre_id=g.genre_id
LEFT JOIN invoice_line as il ON il.track_id=t.track_id;
```

<div class="knitsql-table">

| name            | type  |
| :-------------- | :---- |
| album           | table |
| artist          | table |
| customer        | table |
| employee        | table |
| genre           | table |
| invoice         | table |
| invoice\_line   | table |
| media\_type     | table |
| playlist        | table |
| playlist\_track | table |

Displaying records 1 - 10

</div>

``` sql
---Most sold genres in the USA

WITH USA_tracks AS 
(SELECT il.quantity, il.track_id
FROM invoice_line as il
INNER JOIN invoice as i ON i.invoice_id=il.invoice_id
WHERE i.billing_country="USA")

SELECT g.name as genre_name, SUM(ut.quantity) as quantity_sold
FROM track as t
INNER JOIN USA_tracks as ut ON ut.track_id=t.track_id
INNER JOIN genre as g ON g.genre_id=t.genre_id
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;
```

``` r
genres_prop<-genres%>%
  drop_na(quantity_sold)%>%
  mutate(prop_quantity=quantity_sold/sum(quantity_sold))

plot1<-genres_prop%>%
  ggplot(aes(x=reorder(factor(genre_name), -quantity_sold), y=quantity_sold, fill=genre_name))+
  geom_col()+
  theme_minimal()+
  labs(title = "Most selling genres in the USA",
       y="No. Tracks Sold",
       x="Genre")+
  theme(plot.title = element_text(hjust = 0.5, size = 12, face="bold", 
                                  margin = margin(t = 0, r = 0, b = 15, l = 0)),
        axis.title.y = element_text(margin = margin(t = 0, r = 15, b = 0, l = 0)),
        axis.title.x = element_text(margin = margin(t = 15, r = 0, b = 0, l = 0)),
        legend.position="none")
plot1
```

<img src="Guided-project_Music-store_files/figure-gfm/plot1-1.png" width="100%" height="100%" />
Out of the four artists, it would be wise to select the artists
producing **punk**, **blues** and **Pop** as they are the three best
performing genres out of the four artists, in the USA.

``` sql
--Best performing sales rep/customer support
WITH sales_cus AS (
SELECT e.first_name||" "||e.last_name as sales_rep, e.hire_date, e.birthdate, e.title, c.customer_id
FROM employee as e
INNER JOIN customer as c ON c.support_rep_id = e.employee_id
)

SELECT sc.sales_rep, sc.hire_date, sc.birthdate, sc.title, SUM(i.total) as total_sales
FROM invoice as i
INNER JOIN sales_cus as sc ON sc.customer_id=i.customer_id
GROUP BY 1
ORDER BY 5 DESC;
```

``` r
#format dates

sales$hire_date<-as.numeric(ymd_hms(sales$hire_date, tz="Europe/Berlin"))
sales$birthdate<-as.numeric(ymd_hms(sales$birthdate, tz="Europe/Berlin"))


#Plot performance
plot2<-sales%>%
  ggplot(aes(x=sales_rep, y=total_sales, color=sales_rep))+
  geom_point()+
  theme_minimal()+
  labs(title = "Best performing Sales Rep",
       y="Amount Sold",
       x="Sales Rep")+
  ylim(1000,2000)+
  theme(plot.title = element_text(hjust = 0.5, size = 12, face="bold", 
                                  margin = margin(t = 0, r = 0, b = 15, l = 0)),
        axis.title.y = element_text(margin = margin(t = 0, r = 15, b = 0, l = 0)),
        axis.title.x = element_text(margin = margin(t = 15, r = 0, b = 0, l = 0)),
        legend.position="none")
plot2
```

<img src="Guided-project_Music-store_files/figure-gfm/sales-1.png" width="100%" height="100%" />

``` r
#check correlation between factors
corr <- sales%>%
  select(-1,-4)%>%
  cor(use = "pairwise.complete.obs")


plot3 <- ggcorrplot(
  corr, method = "circle", title = "Factors Influencing Performance")+
  theme(plot.title = element_text(color="black", size=12, face="bold", hjust = 0.5,
                                  margin = margin(t = 0, r = 0, b = 15, l = 0)))
plot3
```

<img src="Guided-project_Music-store_files/figure-gfm/sales-2.png" width="100%" height="100%" />

Jane peacock was the best performing sales support with close to 1750
sold. It would seem that hiring date has a strong negative correlation
(r=-0.95) with total sales. Being hired earlier has a sales support rep
at an advantage over someone that was hired later.

There is also a slight positive correlation (r=0.25) between birth date
and performance. Younger sales support seem to have a slight advantage
over older sales reps. Although this observation is not statistically
significance.

``` sql
--Countries with the best customer value
SELECT c.country, COUNT(DISTINCT c.customer_id) as no_customers, i.invoice_id, SUM(il.unit_price) as total_value, 
       SUM(il.unit_price) / count(distinct c.customer_id) customer_lifetime_value,
       SUM(il.unit_price) / count(distinct il.invoice_id) average_order
FROM customer as c
INNER JOIN invoice as i ON i.customer_id=c.customer_id
INNER JOIN invoice_line as il ON il.invoice_id=i.invoice_id
GROUP BY 1
ORDER BY 2 DESC;
```

``` r
#Recode countries with less than 2 customers 
country<-country%>%
  select(-invoice_id)%>%
  mutate(country=ifelse(no_customers<2,"Others", country))%>%
  group_by(country)%>%
  mutate(no_customers=sum(no_customers))%>%
  mutate(total_value=sum(total_value))%>%
  mutate(customer_lifetime_value=mean(customer_lifetime_value))%>%
  mutate(average_order=mean(average_order))%>%
  distinct()

#Visualize spread
country_longer<-country%>%
  pivot_longer(cols = c(2:5),
               names_to="customer",
               values_to="value")

plot4<-country_longer%>%
 ggplot(aes(x = country, y=value, fill=country))+
  geom_bar(stat = 'identity')+
  theme_minimal()+
  facet_wrap(vars(customer), ncol = 2, scales="free")+
  labs(title="Customer Performance by Country", x = "Country", y = "Customer Performance")+
  theme(plot.title = element_text(hjust = 0.5, size = 12, face="bold", 
                                  margin = margin(t = 0, r = 0, b = 15, l = 0)),
        axis.title.y = element_text(margin = margin(t = 0, r = 15, b = 0, l = 0)),
        axis.title.x = element_text(margin = margin(t = 15, r = 0, b = 0, l = 0)),
        axis.text.x = element_text(size=7.7, angle=-45, vjust = -2),
        axis.ticks = element_blank(),
        legend.position="none")
plot4
```

<img src="Guided-project_Music-store_files/figure-gfm/plot4-1.png" width="100%" height="100%" />

The USA has the most number of customers as well as the highest total
value of orders. However, the Czech Republic has the highest customer
lifetime value, meaning it is a market that one should invest more in
acquiring customers.
