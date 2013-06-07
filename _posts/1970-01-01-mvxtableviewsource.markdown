---
layout: docs
categories: docs
title:  "Choosing an MvvmCross TableViewSource"
---

The available v3 table sources are:

Abstract classes

 - [MvxBaseTableViewSource][1] 
   - base functionality only 
   - no `ItemsSource` - generally not used directly
 - [MvxTableViewSource.cs][2] 
   - inherits from the basetable and addes `ItemsSource` for data-binding
   - inheriting classes need only to implement `protected abstract UITableViewCell GetOrCreateCellFor(UITableView tableView, NSIndexPath indexPath, object item);`

Concrete classes

 - [MvxStandardTableViewSource.cs][3]
   - inherits from `MvxTableViewSource`
   - provides the 'standard iPhone cell types' via `UITableViewCellStyle` 
   - within these you can bind `TitleText`, `DetailText`, `ImageUrl` and (with some teasing) Accessory
 - [MvxSimpleTableViewSource.cs][4]
   - inherits from `MvxTableViewSource`
   - provides a single cell type for all items in the collection - via `string nibName` in the `ctor`
   - within these cells you can bind what you like - see videos (later)
 - [MvxActionBasedTableViewSource.cs][5] 
   - provides some `Func<>`style hooks to allow you to implement `GetOrCreateCellFor` without inheriting a new class from `MvxTableViewSource`

---------

When creating a table I generally choose:

- in demos:
  - a `MvxStandardTableViewSource` - because I get a list without having to create a custom cell
- in real code:
  - a `MvxSimpleTableViewSource` when I only need one cell type
  - a custom class inheriting from `MvxTableViewSource` when I need multiple cell types - e.g. see below

-----

A typical TableSource with multiple cell types typically looks like [PolymorphicListItemTypesView.cs][6]:

    public class PolymorphicListItemTypesView
        : MvxTableViewController
    {
        public PolymorphicListItemTypesView()
        {
            Title = "Poly List";
        }

        public override void ViewDidLoad()
        {
            base.ViewDidLoad();

            var source = new TableSource(TableView);
            this.AddBindings(new Dictionary<object, string>
                {
                    {source, "ItemsSource Animals"}
                });

            TableView.Source = source;
            TableView.ReloadData();
        }

        public class TableSource : MvxTableViewSource
        {
            private static readonly NSString KittenCellIdentifier = new NSString("KittenCell");
            private static readonly NSString DogCellIdentifier = new NSString("DogCell");

            public TableSource(UITableView tableView)
                : base(tableView)
            {
                tableView.RegisterNibForCellReuse(UINib.FromName("KittenCell", NSBundle.MainBundle),
                                                  KittenCellIdentifier);
                tableView.RegisterNibForCellReuse(UINib.FromName("DogCell", NSBundle.MainBundle), DogCellIdentifier);
            }

            public override float GetHeightForRow(UITableView tableView, NSIndexPath indexPath)
            {
                return KittenCell.GetCellHeight();
            }

            protected override UITableViewCell GetOrCreateCellFor(UITableView tableView, NSIndexPath indexPath,
                                                                  object item)
            {
                NSString cellIdentifier;
                if (item is Kitten)
                {
                    cellIdentifier = KittenCellIdentifier;
                }
                else if (item is Dog)
                {
                    cellIdentifier = DogCellIdentifier;
                }
                else
                {
                    throw new ArgumentException("Unknown animal of type " + item.GetType().Name);
                }

                return (UITableViewCell) TableView.DequeueReusableCell(cellIdentifier, indexPath);
            }
        }
    }

------------

TODO - below here is separate article?

For examples of creating custom tables and cells:

- there's a lot of demos of table use in the N+1 series - indexed at http://mvvmcross.wordpress.com

 - N=2 and N=3 are very basic
 - N=6 and N=6.5 covers a book list (a good place to start)
 - N=11 covers collection views
 - N=12 to N=17 make a large app with a list/table from a database

- the "Working with Collections" sample has quite a lot of Table and List code - https://github.com/slodge/MvvmCross-Tutorials/tree/master/Working%20With%20Collections

- tables are used during the Evolve presentation - http://xamarin.com/evolve/2013#session-dnoeeoarfj

- there are other samples available - see https://github.com/slodge/MvvmCross-Tutorials/ (or search on GitHub for mvvmcross - others are also posting samples)


  [1]: https://github.com/slodge/MvvmCross/blob/v3/Cirrious/Cirrious.MvvmCross.Binding.Touch/Views/MvxBaseTableViewSource.cs
  [2]: https://github.com/slodge/MvvmCross/blob/v3/Cirrious/Cirrious.MvvmCross.Binding.Touch/Views/MvxTableViewSource.cs
  [3]: https://github.com/slodge/MvvmCross/blob/v3/Cirrious/Cirrious.MvvmCross.Binding.Touch/Views/MvxStandardTableViewSource.cs
  [4]: https://github.com/slodge/MvvmCross/blob/v3/Cirrious/Cirrious.MvvmCross.Binding.Touch/Views/MvxSimpleTableViewSource.cs
  [5]: https://github.com/slodge/MvvmCross/blob/v3/Cirrious/Cirrious.MvvmCross.Binding.Touch/Views/MvxActionBasedTableViewSource.cs
  [6]: https://github.com/slodge/MvvmCross-Tutorials/blob/master/Working%20With%20Collections/Collections.Touch/Views/Samples/PolymorphicListItemTypesView.cs
  
