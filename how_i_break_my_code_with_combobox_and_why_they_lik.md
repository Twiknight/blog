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
  private string selectedCity;
  private string selectedArea;
  
  public string SelectedCity
  {
    get 
    {
      return selectedCity;
    }
    set
    {
      selectedCity = value;
      NotifyPropertyChanged("SelectedCity");
    }
  }
  
  public string SelectedArea
  {
    get
    {
      return selectedArea;
    }
    set
    {
      selectedArea = value;
      NotifyPropertyChanged("SelectedArea");
    }
  }
    
  public IEnumerable<string> Areas
  {
    get 
    {
      return cityBook.Values.Distinct();
    }
  }
  
  public IEnumerable<City> Cities
  {
    get
    {
      return cityBook.Where(i => i.Value == SelectedArea)
                  .Select(i => i.Key);
    }
  }
  //Event notifiers and other methods ....
  ...
}
```
Let me make a brief explanation about what I did:
 1. I extracted the data to a `Dictionary`, setting city name as `Key` and area name as `Value`;
 2. I generated the fields for holding selected values;
 3. I generated the display data set for Combo Box items-binding: 
    One for all the distinct Areas, the other for all the cities filtered by selected area;
    
But, the above code won't work at all, 
because when you select an area, WPF won't automatically inform the change on `Cities`.

Though it looks somehow wired, I add it to setter of `SelectedArea`.
```csharp
  public string SelectedArea
  {
    get { ... }
    set
    {
      selectedArea = value;
      NotifyPropertyChanged("SelectedArea", "Cities");
    }
  }
```
Here I was fully convinced it won't cause trouble, since each select on area is sure to trigger a change on city.

If I stopped here, it would be fine.  
But one day, my PM came to me,  
 "Hey, tk. Our customers enjoy your city-area selectors a lot, except they want to add a small feature."  
 "So..What do they want?"  
 "Can you make the city-drop filled with all the cities if no area is selected?"  
 "Sure, that's easy!" 
 To make it, I just need to replace `i => i.Value == SelectedArea` with `i => i.Value.Contains(SelectedArea)`.  
 "Good, and when they select a city directly from the city-drop, the area-drop should show the correct area."
 
It is still easy, right?
When we select city, just set the area as well, :)

```csharp
public string SelectedCity
  {
    get { ... }
    set
    {
      selectedCity = value;
      NotifyPropertyChanged("SelectedCity");
      
      if (string.IsNullOrEmpty(value))
      {
          return;
      }
      string currentArea = string.Empty;
      cityBook.tryGetValue(value, out currentArea);
      SelectedAera = currentArea;
    }
  }
```
Now you may feel some bad smell. 
I'm copying myself in the setter, and the code is really unreadable.

And it would cause stack overflow if change city from "Shanghai" to "Nanjing".

Why? 
A `ComboBox` would try to reset the `SelectedItem` binding to it, whenever a property change on its binding `ItemSource` is detected.

See, I had to know what's going on inside the `ComboBox` Component,
if I want to make it work.

Actually, following lines will work:
```csharp
set
{
  if (string.IsNullOrEmpty(value))
  {
      return;
  }

  string currentArea = SelectedArea;
  cityBook.TryGetValue(value, out currentArea);
  if (currentArea != SelectedArea)
  {
      SelectedArea = currentArea;
  }

  selectedCity = value;
  NotifyPropertyChanged("SelectedCity");
}
```

But it's twisted. As well, I need to know the details of `ComboBox`.