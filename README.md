# yieldmo_sql1
pull out all usernames of people who had an active subscription during Q4 2018 and are not active now:

# Define the DDL/schema to hold user, product and subsctiption data

    CREATE TABLE IF NOT EXISTS `user` (
      `user_id` int(6)  NOT NULL,
      `login_id` int(3) NOT NULL unique,
      `email` varchar(200) ,
      `full_name` varchar(200),
      `create_date` timestamp,
      PRIMARY KEY (`user_id`)
    ) DEFAULT CHARSET=utf8;

    CREATE TABLE IF NOT EXISTS `product` (
      `product_id` int(6)  NOT NULL,
      `product_name` varchar(200) ,
      `product_description` varchar(500) ,
      PRIMARY KEY (`product_id`)
    ) DEFAULT CHARSET=utf8;

    CREATE TABLE IF NOT EXISTS `subscription` (
      `subscription_id` int(6) NOT NULL,
      `user_id` int(6)  NOT NULL,
      `product_id` int(6) NOT NULL,
      `subscription_startdate` timestamp ,
      `subscription_enddate` timestamp,
      PRIMARY KEY (`subscription_id`)
    ) DEFAULT CHARSET=utf8;
    
# Insert values into the above defined schemas

    INSERT INTO `user` (`user_id`, `login_id`, `email`, `full_name`, `create_date`) VALUES
      ('1000', '001', 'u001@email.com', 'aeron targerian', '2013-05-19'),
      ('2000', '002', 'u002@email.com', 'robert baratheon', '2015-08-20'),
      ('3000', '003', 'u003@email.com', 'eddard stark', '2016-07-20'),
      ('4000', '004', 'u004@email.com', 'rhegar targerian', '2018-03-15'),
      ('5000', '005', 'u005@email.com', 'john snow', '2018-02-15'),
      ('6000', '006', 'u006@email.com', 'tyrion lannister', '2018-08-15'),
      ('7000', '007', 'u007@email.com', 'walder frey', '2018-06-15'),
      ('8000', '008', 'u008@email.com', 'Aarya Stark', '2018-07-15')
     ;

    INSERT INTO `product` (`product_id`, `product_name`, `product_description`) VALUES
      ('901', 'starledger', 'starledge daily print news'),
      ('902', 'heraldnews', 'heraldnews daily print news'),
      ('903', 'ny times', 'ny time daily print news'),
      ('904', 'washington post', 'washington post daily print news'),
      ('905', 'chicago tribune', 'chicago tribune daily print news'),
      ('906', 'boston herald', 'boston herald daily print news')
     ;

    INSERT INTO `subscription` (`subscription_id`, `user_id`, `product_id`, `subscription_startdate`, `subscription_enddate`) VALUES
      ('801', '1000', '901', '2013-05-19', '2013-08-15'),
      ('802', '1000', '901', '2014-05-19', '2014-08-15'),
      ('803', '1000', '901', '2015-05-19', '2015-08-15'),
      ('805', '1000', '901', '2018-05-19', '2018-09-15'),
      ('806', '2000', '901', '2018-05-19', '2018-08-15'),
      ('807', '2000', '902', '2018-05-19', '2018-08-15'),
      ('808', '2000', '902', '2017-05-19', '2017-08-15'),
      ('809', '2000', '903', '2018-05-19', '2018-06-15'),
      ('810', '2000', '903', '2019-05-01', '2019-06-30'),
      ('811', '3000', '902', '2018-05-01', '2019-06-30'),
      ('812', '4000', '905', '2018-04-01', '2018-09-30')
      ;
  # Lets manually identify and seperate the records from subsciption table that are active in Q3 2018
  
  
      ('805', '1000', '901', '2018-05-19', '2018-09-15'),
      ('806', '2000', '901', '2018-05-19', '2018-08-15'),
      ('807', '2000', '902', '2018-05-19', '2018-08-15'),
      ('812', '4000', '905', '2018-04-01', '2018-09-30')
   
# Lets lets take the above narrowed down list and compare it against with subscription data to see if any of them are active now. We will ignore the active ones and list the ones that are currently not active
   
        ('805', '1000', '901', '2018-05-19', '2018-09-15'),
        ('812', '4000', '905', '2018-04-01', '2018-09-30')
# We will now note down the user_ids and do a lookup on `user` table to list the users full_names 
   
        user_id : 1000 --> full_name : aeron targerian
        user_id : 4000 --> full_name : rhegar targerian
        
# Now we will automate the process through  a SQL script. 
### SQL starts from subsctiption table and  performing a left join user table. The first condition is to limit the records to users who had a subscription in Q3 2018 and the second condition is to verify if any of those users currently do not have an active subscription. The second condition is met by performing a lookup on the results of a subquery.

        select distinct(l.full_name) 
        from subscription k left join user l 
        on 
            k.user_id = l.user_id 
        where
            (extract(YEAR FROM k.subscription_startdate) = 2018 and extract(QUARTER from k.subscription_startdate) = 3)
            or
            (extract(YEAR FROM k.subscription_enddate) = 2018 and extract(QUARTER from k.subscription_enddate) = 3)
        and k.user_id not in
            (select a.user_id from user a , subscription c 
            where a.user_id = c.user_id  
            and current_date  between c.subscription_startdate and c.subscription_enddate
            )
 
 # Output of the SQL run
 
        full_name
        --------------
        aeron targerian
        rhegar targerian
   
   
   
  
   
 
  
