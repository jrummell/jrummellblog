# Converting DateTime to String and back


I spent about 20 frustrating minutes the other day wondering why a sql  
 query wasn't selecting the record I wanted. Everything looked right  
 until I stepped through my code the 3rd time. Then I discovered my  
 problem.

In a web form, I allow a user to select a row in a  
 GridView that fires an event to populate a DetailsView using an  
 ObjectDataSource. The select method of the ObjectDataSource takes two 
 parameters, a DateTime and a string. Since the date is coming from the 
 GridView, I was just using Convert.ToDateTime([date cell].ToString()).

I   discovered that the DateTime displayed in the GridView was '5/21/2007 8:51:42 AM' while the DateTime in the database was '2007-05-21 08:51:42.153'. They're close, but not exactly the same. It's that  
 missing fraction of a second that made my where clause incorrect.

So   then I thought, "How can I successfully convert a DateTime to a string   and back?" After a short pause it hit me, "Ticks". No, not the kind of [ticks](http://en.wikipedia.org/wiki/Tick) I'm afraid of getting when backpacking in woods, but [DateTime.Ticks](http://msdn2.microsoft.com/en-us/library/system.datetime.ticks.aspx). I used a [HiddenField](http://msdn2.microsoft.com/en-us/library/system.web.ui.webcontrols.hiddenfield%28vs.80%29.aspx)
to store the string representation of the selected DateTime in ticks,  
 and then added an overloaded select method that takes a ticks (long)  
 parameter. In the new method I simply construct a DateTime from the  
 ticks and call the original select method.

So in summary, to convert from DateTime to string and back, use ticks.


