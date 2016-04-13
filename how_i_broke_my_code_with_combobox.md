# How I break my code with Combo Box

### TL;DR
 This is a story about my experience of anti-pattern programming. And I'll share my opinion on how to avoid such mistakes.

### Story: the nightmare with Combo Box

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
Actually, the problem I faced was far more complex than the case above.
After a few months, they became lines-only-god-knows.

It is weird !  

### Let's what is going wrong with my code

When practicing OOP, it is better to stay ignorant than omniscient.

When embedding the code of updating `Cities` into the setter of `SelectedArea`, 
I was one step close to trouble.
That was the first step to abuse the rule of Single Responsibility. 
It was OK, though, if the complexity of the your logic would stay that level.

But, for more complex cases, that's a typical anti-pattern. 
Obviously, A setter is expected handle affairs related with the private field it will set. Say, in my case, it was to notify the value change of `SelectedArea`.

Then how about the `Cities`, should `SelectedArea.set` know what will happen on other properties? Theoretically, it shouldn't. Have you ever heard that a setter should take the responsibility as a Dependency Manager?

Though, it won't hurt if your logic is simple enough. When things go complicated or the requirements change, 
following the existing code style (most developers do this when they maintain legacy code bases) will mess up the project.

That's what happened when I add a new feature to my Combo Boxes.
Operating unrelated fields in a setter is kind of Black Magic. No one but you know what happen if you "set" a property. And I believe you'll forget what you did here months later  : )

What's worse, random property assigning makes it nearly impossible to analyze your program. When something unexpected happens, only step-by-step debug can save you. When the pm requires new feature or modification, the legacy code will be a new nightmare. You never know if it will crash somewhere beyond your imagination when you try to modify a line.

### Easy to delete or Easy to extend?

Some developers may hold a misunderstanding of the Open-Close Principle. They may think, a well-designed program should avoid any modification on existing code. Actually that's impossible, unless your load every single line of your code from Database :). 

To some degree, Maintainable code is code easy to delete.
OCP only helps when you can predict a certain kind of extension. 
It doesn't mention how to handle predictable modifications and discards.
But we often need to handle such cases for some business reason.
That's the occasion where it is necessary to get hands dirty.

But how to define if a line is easy to delete or not?
In my opinion, when it is the time you delete or replace one or a couple lines, you know what will happen.
Say, deleting a line will crash the whole system, but you know who the system would break. 
You know where an exception or error would raise, how it effects other parts of the system, and the consequence of the crash.
Moreover, if you replace with one or a few lines, you don't need to modify multiple code blocks.
This is important, modifying one part is better than multiple in most cases.

Maybe you've found, write something easy to delete is not on the opposite of OCP.
It is supplement to OCP, it handles with things OCP didn't tell us. 
([here is something interesting to read about delete and extend](http://programmingisterrible.com/post/139222674273/write-code-that-is-easy-to-delete-not-easy-to))

### Back to my Combo Boxes

If I say here, I could avoid the mess when I wrote the first line with my Combo Boxes.
 I must be lying.

Actually, I even wouldn't what would happen when I slightly sacrificed the Single Responsibility Principle to my convenience.
But I had a chance to make things work well when implementing the new requirements.
A refactory on how I handle the dependencies of different properties would totally avoid such a pitfall. Thus my code will be easier to delete.

__The main problem I faced was that Properties (or Combo Boxes) have twisted relationship.__ 
It forced me to take care of how they related each other in the setter.

However, Let's consider the nature of GUI. GUI works in two way:
 1. Present data;
 2. Accept user input;

__We can say that, what presented on GUI reflects the state of a data set;
 Then user inputs changes the state of the data set.__

It is totally nothing to do with the relationship of the dependencies.
The GUI  program is like one mapping from the collection of data to a collection of pixels.
That is, if I have knowledge of the data, I should know what the view displays.
That's why we create getters for binding props.

Then the setters, they accept user inputs and map them back to data.
That's it, should I handle the relationship in the setters?
If I treat the setters as operators on data rather than property setter, it is OK. 
But if I do so, I should not notify changes inside the setters.




