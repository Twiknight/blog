# How I break my code with Combo Box and why they like Redux

In my short period working with WPF,
the worst nightmare that haunted me was handling a Combo Box.
If there were things worse, that must be handling a couple of Combo Box.

Here, I'm to share a story how I made my code base a mess with two Combo Boxes. 
 I think that experience my have something to do with the fashion of using Redux.

Say, you have a like of Object to show in two Combo Boxes.
To make it clearer, imagine you have a list of cities, 
 which may be the destinations of mails.

Each city is described with the following structure:
```csharp
public struct City
{
  public string Name { get; set;}
  public string Area { get; set;}
}
```

Then you're told to show those cities with two Combo Box:
 - The first shows the areas;
 - While the second shows the city names.

It's easy to see that the two Combo Boxes have some relationship on the data their display.
When you select a area, it will set a filter on the city selector.

That is soooooo easy! Right? 
Then I finished it without any effort:

```csharp
public class ViewModel : INotifyPropertyChanged
{
  private Dictionary<string, string> cityBook;
  private City selectedCity;
  private string selectedCountry;
  
  public IEnumerable<string> Countries
  {
    get 
    {
      return cityBook.values;
    }
  }
  public List<City> Cities{}
}
```


