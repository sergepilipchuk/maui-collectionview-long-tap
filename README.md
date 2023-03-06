# List Cards

This example shows you how to use the list card to preview information from a set of related items or objects. 

<img width="50%" src="https://user-images.githubusercontent.com/12169834/223110944-4904bf34-da91-4685-9656-fb7e09905d42.png"/>

## Used Controls and Their Properties

**A. Header**

  * [Label](https://learn.microsoft.com/en-us/dotnet/maui/user-interface/controls/label?view=net-maui-7.0): [Text](https://learn.microsoft.com/en-us/dotnet/api/microsoft.maui.controls.label.text?view=net-maui-7.0)
  * [SimpleButton](https://docs.devexpress.com/MAUI/DevExpress.Maui.Controls.SimpleButton): [Command](https://docs.devexpress.com/MAUI/DevExpress.Maui.Controls.SimpleButton.Command)

**B. List Items**

  * [Label](https://learn.microsoft.com/en-us/dotnet/maui/user-interface/controls/label?view=net-maui-7.0): [Text](https://learn.microsoft.com/en-us/dotnet/api/microsoft.maui.controls.label.text?view=net-maui-7.0)
  * [SimpleButton](https://docs.devexpress.com/MAUI/DevExpress.Maui.Controls.SimpleButton): [Command](https://docs.devexpress.com/MAUI/DevExpress.Maui.Controls.SimpleButton.Command), [CommandParameter](https://docs.devexpress.com/MAUI/DevExpress.Maui.Controls.SimpleButton.CommandParameter)

**C. Footer (optional)**

  * [SimpleButton](https://docs.devexpress.com/MAUI/DevExpress.Maui.Controls.SimpleButton): [Command](https://docs.devexpress.com/MAUI/DevExpress.Maui.Controls.SimpleButton.Command)
## A. Header

### Anatomy
The header area displays a list header that you can click to opens detail list.

<img width="50%" src="https://user-images.githubusercontent.com/12169834/223118601-0386f5c7-fd4c-4fe1-a03d-775b5e12a9e6.png"/>

### Behavior

Follow the steps below to open the full list of items on the header click:

1. Call the [Routing.RegisterRoute](https://learn.microsoft.com/en-us/dotnet/api/microsoft.maui.controls.routing.registerroute?view=net-maui-7.0#microsoft-maui-controls-routing-registerroute(system-string-system-type)) method to register the page that contains the full list of items:

    ```csharp
    public partial class App : Application {
        public App() {
            InitializeComponent();
            Routing.RegisterRoute("completeList", typeof(CompleteListPage));
            MainPage = new AppShell();
        }
    }
    ```

2. Specify the [SimpleButton.Command](https://docs.devexpress.com/MAUI/DevExpress.Maui.Controls.SimpleButton.Command) property to define the header click command:

    ```xaml
    <VerticalStackLayout >
        <dxco:SimpleButton Text="{Binding Title}" Command="{Binding NavigateToAllCommand}" ...>
            <dxco:SimpleButton.Content>
                <Grid>
                    <Label Text="{Binding Title}" .../>
                    <Label Text="{Binding Path=Items.Count, StringFormat='All ({0})'}" .../>
                </Grid>
            </dxco:SimpleButton.Content>
        </dxco:SimpleButton>
    </VerticalStackLayout>
    ```
    
    ```csharp
    namespace CollectionViewWithActionButtons.ViewModels {
        // ...
        public class Card : BindableBase {
            public Card(string title) {
                // ...
                NavigateToAllCommand = new Command(NavigateToAll);
            }
            // ...
            public ICommand NavigateToAllCommand { get; }
            
            public async void NavigateToAll() {
                var navigationParameter = new Dictionary<string, object> { { "Parent", this } };
                await Shell.Current.GoToAsync("completeList", navigationParameter);
            }
        }
    }
    ```

    The **NavigateToAll** method calls the [GoToAsync](https://learn.microsoft.com/en-us/dotnet/api/microsoft.maui.controls.shell.gotoasync?view=net-maui-7.0#microsoft-maui-controls-shell-gotoasync(microsoft-maui-controls-shellnavigationstate-system-collections-generic-idictionary((system-string-system-object)))) method to open the detail view.

3. Specify the [QueryProperty](https://learn.microsoft.com/en-us/dotnet/api/microsoft.maui.controls.querypropertyattribute?view=net-maui-7.0) attribute for the **CompleteListViewModel** class to pass the clicked header context to the command registered in the previous step:
   
   ```csharp
   namespace CollectionViewWithActionButtons.ViewModels {
       [QueryProperty(nameof(ParentCard), "Parent")]
       internal class CompleteListViewModel : INotifyPropertyChanged {
           public Card parentCard;
           public Card ParentCard {
               get { return parentCard; }
               set {
                   parentCard = value;
                   OnPropertyChanged();
               }
           }
           public event PropertyChangedEventHandler PropertyChanged;
           void OnPropertyChanged([CallerMemberName] string propertyName = null) {
               PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
           }
       }
   }
   ```

## B. List Items


This area contains list of items that you can open (click on it) or remove (the **✕** button on the right). 


<img width="50%" src="https://user-images.githubusercontent.com/12169834/223119451-8b5fb385-590c-4707-a05d-8dbd662a23cc.png"/>

### Behavior

Follow the steps below to open the **CollectionView** item detailed information on click:

1. Specify the [SimpleButton.Command](https://docs.devexpress.com/MAUI/DevExpress.Maui.Controls.SimpleButton.Command) and [SimpleButton.CommandParameter](https://docs.devexpress.com/MAUI/DevExpress.Maui.Controls.SimpleButton.CommandParameter) propertes to define the item click command. The following code sample uses the [FindAncestorBindingContext](https://learn.microsoft.com/en-us/dotnet/api/microsoft.maui.controls.relativebindingsourcemode?view=net-maui-7.0) binding to get the command of the parent object's **ItemClick** and **Hide** commands:
   
    ```xaml
    <dxco:SimpleButton Command="{Binding Source={RelativeSource Mode=FindAncestorBindingContext,
                    AncestorType={x:Type viewModels:Card}}, Path=ItemClickCommand}"
                    CommandParameter="{Binding}">
        <Grid ...>
            <Image Source="{Binding Icon}" .../>
            <Label Text="{Binding Name}" .../>
        </Grid>
    </dxco:SimpleButton>
    <dxco:SimpleButton Text="&#x2715;" Command="{Binding Source= {RelativeSource Mode=FindAncestorBindingContext,
                    AncestorType={x:Type viewModels:Card}}, Path=HideCommand}" CommandParameter="{Binding}" .../>
    ```

1. Define the **Card** class's **ItemClick** and **Hide** commands:
   
    ```csharp
    namespace CollectionViewWithActionButtons.ViewModels {
        public class ViewModel : BindableBase {
            public ViewModel() {
                Cards = DataGenerator.CreateCards();
            }
            public ObservableCollection<Card> Cards { get; set; }
        }
        
        public class Card : BindableBase {
            public Card(string title) {
                Title = title;
                HideCommand = new Command<CardItem>(HideItem);
                ItemClickCommand = new Command<CardItem>(ItemClick);
                // ...
            }

            public string Title { get; }
            public ICommand HideCommand { get; }
            public ICommand ItemClickCommand { get; }
            // ...
            public async void ItemClick(CardItem clickedItem) {
                if (clickedItem == null) return;
                await Application.Current.MainPage.DisplayAlert("Item Click", clickedItem.Name, "OK");
            }
            // ...
        }

        public class CardItem : BindableBase {
            public string Name { get; set; }
            public string Subtitle { get; set; }
            public DateTime CreatedDate { get; set; }
            public ImageSource Icon { get; set; }
        }

        public class BindableBase : INotifyPropertyChanged {
            public event PropertyChangedEventHandler PropertyChanged;
            protected void NotifyPropertyChanged([CallerMemberName] string propertyName = "") {
                PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
            }
        }
    }
    ```

## C. Footer (Optional)

The footer area includes buttons that hide or apply all the items from this group. 

<img width="50%" src="https://user-images.githubusercontent.com/12169834/223119520-25279691-b6cd-4147-b3aa-8154d0b9670e.png"/>

### Behavior

Buttons within the footer are visible only when the **Card** class's **AllowCommonActions** property is `true`. The **PrimaryActionName** and **SecondaryActionName** properties define button names. 

The following code snippet specifies whether the **Security** card's footer buttons are visible and define their names:

```csharp
public static class DataGenerator {
    public static ObservableCollection<Card> CreateCards() {
        ObservableCollection<Card> cards = new ObservableCollection<Card>();
        cards.Add(new Card("Security") {
            Items = new ObservableCollection<CardItem>() {
                // ...
            },
            PreviewItemsCount = 4,                
            AllowCommonActions = true,
            PrimaryActionName = "Dismiss All",
            SecondaryActionName = "Apply All"
        });
        cards.Add(new Card("Performance") {  
            Items = new ObservableCollection<CardItem>() {
                // ...
            },
            PreviewItemsCount = 3,
        });
        // ...
    }
    // ...
}
```


Follow the steps below to implement commands that performs operations over all **CollectionView** items within the card:

1. Specify the [SimpleButton.Command](https://docs.devexpress.com/MAUI/DevExpress.Maui.Controls.SimpleButton.Command) property to define the click commands for both footer buttons. These commands runs only when the 
   
    ```xaml
    <HorizontalStackLayout HorizontalOptions="End" x:Name="commonActionsPanel" Padding="20,5,20,0">
        <HorizontalStackLayout.Triggers>
            <DataTrigger TargetType="HorizontalStackLayout" Binding="{Binding AllowCommonActions}" Value="False">
                <Setter Property="IsVisible" Value="False"/>
            </DataTrigger>
        </HorizontalStackLayout.Triggers>
        <dxco:SimpleButton Text="{Binding PrimaryActionName}" Command="{Binding SecondaryActionCommand}" .../>
        <dxco:SimpleButton Text="{Binding SecondaryActionName}" Command="{Binding PrimaryActionCommand}" .../>
    </HorizontalStackLayout>
    ```

1. Define the **Card** class's **PrimaryAction** and **SecondaryAction** commands:

    ```csharp
    namespace CollectionViewWithActionButtons.ViewModels {
        // ...
        public class Card : BindableBase {
            public Card(string title) {
                // ...
                PrimaryActionCommand = new Command(PrimaryAction);
                SecondaryActionCommand = new Command(SecondaryAction);
            }
            // ...
            public ICommand PrimaryActionCommand { get; }
            public ICommand SecondaryActionCommand { get; }
            public bool AllowCommonActions { get; set; }
            public string PrimaryActionName { get; set; }
            public string SecondaryActionName { get; set; }
            // ...
            }
            // ...
            public async void PrimaryAction() {
                await Application.Current.MainPage.DisplayAlert("Primary Action", "Click", "OK");
            }
            public async void SecondaryAction() {
                await Application.Current.MainPage.DisplayAlert("Secondary Action", "Click", "OK");
            }
            // ...
        }
    }
    ```
## Files to Review

## Documentation

## More Examples
