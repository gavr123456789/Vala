# DnD

Тут разбор вот [этого](https://gitlab.com/doublehourglass/dnd_order_list_box) кастомного виджета 

{% tabs %}
{% tab title="ListBox" %}
```csharp
using Gtk;
using Gdk;
namespace DHGWidgets {

/**
 * "Список заказов Dnd" - это / / Gtk.ListBox / / потомок, позволяющий переупорядочивать его строки
 * with pointer or keyboard actions.
 * Contrary to some other implementations that use child widgets to enable drag'n'drop behavior
 * the ''DndOrderListBox'' is widget agnostic and operates only on native rows.
 * The only caveat here is that child widgets must be able to receive drag events
 * and thus must have own X11 windows.
 *
 * = Limitations =
 *
 * The ''DndOrderListBox'' construction methods allow it to be populated only with
 * a //GLib.ListModel// descendant (most likely //GLib.ListStore//).
 * However, as this component is a //work in progress// it may be extended
 * to accept common //Gtk.ListBox// add, remove and insert methods
 * at a later date.
 * There is neither way nor sense to enable multiselection abilities
 * in this particular widget. So, they remain masked.
 *
 * = Keyboard bindings =
 *
 * This widget provides users with extended keyboard bindings. Apart from standard
 * //Gtk.ListBox// bindings (that are remapped) it comes with Vim-like ones by default.<<BR>>
 * Bindings are as follows:
 *
 * == //moving selection// ==
 *
 * ''up'' and ''down'' cursors<<BR>>
 * ''k'' and ''j'' letters (//vim//)
 *
 * == //initiating keyboard "drag"// ==
 *
 * ''<Ctrl>up'' and ''<Ctrl>down''
 * ''i'' letter (//vim//)
 *
 * == //moving "drop" marker// ==
 *
 * ''<Ctrl>up'' and ''<Ctrl>down'' (so, to move marker <Ctrl> must stay depressed, it's like this to
 * be consistent with native //Gtk.ListBox// behavior of these keys)
 * ''k'' and ''j'' letters (//vim//)
 *
 * == //confirming// ==
 *
 * ''Enter'' key
 *
 * == //interrupting// ==
 *
 * ''Escape'' key or pointer d'n'd action
 *
 */

public class DndOrderListBox : ListBox {

// The cached CssProvider.
static CssProvider _provider = new CssProvider ();

// The identifier of data that is transferred during pointer drag.
const string DATA_ID = "DND_ORDER_LIST_BOX_ROW_DATA";
// The default CSS stylesheet resource location.
const string DEFAULT_CSS_RESOURCE = "/me/dhg/dndorderlistbox/DndOrderListBox.css";

/**
 * CSS classes that are used to change appearance of rows and show drag indicator.
 * Left as //public// for convenience.
 */
public static string[] mgr_css_classes {get; set; default = {"place_up", "place_dn", "dragged", "dragging"};}

// Data must somehow be transferred on drop via pointer. While all could
// be accomplished via DndMgr, defining universal way to do it may allow
// for acceptance of drops from outside this widget.
static TargetEntry[] entries = {
	TargetEntry () {
		target = DATA_ID, flags = TargetFlags.SAME_APP, info = 0
		}
	};

// Private field storing reference to DndMgr
DndMgr _dnd_mgr;

/**
 * Stores a model reference for easier access, as parent has it //bound_model//
 * boxed and thus private.
 */
// <caveat> There is also a Gtk.ListStore so care is advised. That's why it's
// declared with prefix.
public GLib.ListStore model {get; set;}

/**
 * Stores a reference to the keyboard bindings initializer. This class
 * takes care about overriding and setting new bindings.
 */
public static DndOrderListBoxBindings kbd_bindings {get; set;}

static construct {
	// The below method must be run during class registration to allow the widget being identified by
	// a string element name in css selectors.
	set_css_name ("dnd-order-list-box");
	// Load default css here and add it to the Screen.
	_provider.load_from_resource(DEFAULT_CSS_RESOURCE);
	StyleContext.add_provider_for_screen (Screen.get_default(), _provider, STYLE_PROVIDER_PRIORITY_APPLICATION);
	// And last, set up keybinding.
	kbd_bindings = new DndOrderListBoxBindings(BindingSet.by_class((ObjectClass) ( typeof ( DndOrderListBox )).class_ref()));
	}
/**
 * Plain constructor. Mind to attach a model with bind_model method.
 */
public DndOrderListBox() {}
/**
 * This construction method initializes ''DndOrderListBox'' with a //GLib.ListModel//
 * derivative.
 * Parameters are the same as in //Gtk.ListBox.bind_model//
 */
public DndOrderListBox.with_list(GLib.ListStore list_model, ListBoxCreateWidgetFunc fn) {
	// Why this way, not via Object call?
	// ListBoxCreateWidgetFunc is a delegate and Vala doesn't like copying
	// it so it's not allowed in GObject type constructor -
	// otherwise it could be stored and managed via own peoperties.
	bind_list(list_model, fn);
	}
construct {
	// Only DndMgr is initialized here as it's common for both construction
	// calls available.
	_dnd_mgr = new DndMgr(this);
	}


// Utility method that initiates keyboard action if it isn't yet initiated.
void kbd_init () {
	// keyboard driven operation is made up of two phases
	// any pointer drag operation will cancel it when initiated between said
	// phases therefore we must make sure the keyboard driven operation
	// is initiated
	if ( _dnd_mgr.keyboard_action != true ) {
		_dnd_mgr.keyboard_drag(get_selected_row());
		}
	}
// Utility method that exchanges entries in the model effectively
// rearranging rows.
// An additional benefit is keeping a reference to the source object,
// so saved another line otherwise sacrificed for a variable.
void exchange_rows(int remove, int add, Object? source) {
	model.remove(remove);
	model.insert(add, source);
	}
// It took me a while to figure out this [Signal (action = true)] as signals
// in vala have implicit constructors with no obvious place to put flags.
// it turns out that signal parameters can be set via attributes
// for clarity, signals triggered via BindingEntry must be have
// G_BINDING_ACTION flag set to avoid being them managed as usual signals.
/**
 * Manage starting keyboard action in Vim style.
 */
[Signal (action = true)]
public virtual signal void kbd_vim () {
	if ( _dnd_mgr.keyboard_vim_mode ) {
		_dnd_mgr.dragged(false);
		} else {
		kbd_init();
		_dnd_mgr.keyboard_vim_mode = true;
		}
	}
/**
 * Manage moving "drag" indicator when the keyboard action is pending.
 */
[Signal (action = true)]
public virtual signal void kbd_move_indicator (bool dir) {
	kbd_init();
	_dnd_mgr.keyboard_move_mark(dir);
	}
/**
 * Manage moving cursor (selection) by keyboard. If the vim mode is active
 * it calls //kbd_move_indicator//.
 */
[Signal (action = true)]
public virtual signal void kbd_move_cursor (MovementStep step, int count) {
	// If keyboard action is off just move the cursor normally.
	if ( _dnd_mgr.keyboard_action != true ) {
		move_cursor(step, count);
		}
	// If vim mode is on and movement step indicates single line movement
	// (DISPLAY_LINES) call //kbd_move_indicator//.
	if ( _dnd_mgr.keyboard_vim_mode && step == MovementStep.DISPLAY_LINES ) {
		kbd_move_indicator(count != 1);
		}
	}
/**
 * Interrupts the keyboard action.
 */
[Signal (action = true)]
public virtual signal void kbd_break () {
	if ( _dnd_mgr.keyboard_action == true ) {
		_dnd_mgr.dragged(false);
		}
	}
/**
 * Confirms "drag" by keyboard action and moves the item dragged to a new place.
 */
[Signal (action = true)]
public virtual signal void kbd_confirm () {
	if ( _dnd_mgr.keyboard_action == true ) {
		// Cancel the keyboard action and remove styling
		// (all done by _dnd_mgr.dragged).
		_dnd_mgr.dragged(false);
		// Remove source row from the list (cache it so a reference remains)
		// and insert it at a new position.
		exchange_rows(_dnd_mgr.source_position, _dnd_mgr.target_position, model.get_item(_dnd_mgr.source_position));
		}
	}

/**
 * This method binds external model with the ''DndOrderListBox''.<<BR>>
 * It also initializes internal operations that are dependent on the model.
 */
public void bind_list (GLib.ListStore list_model, ListBoxCreateWidgetFunc fn) {
	// First push the model to a property.
	model = list_model;
	// Then bind model to the DndOrderListBox using provided creation function using
	// parent's class method (as it's masked on this level).
	base.bind_model(model, ( obj ) => {return fn (obj);});
	// Then get a number of model items needed later to clamp operations.
	// Update the drag manager so keyboard operations won't get too far.
	_dnd_mgr.list_max = model.get_n_items();
	// Then connect to changes in the model to update the state of this widget.
	model.items_changed.connect(items_changed_cb);
	// Finally get every new row and pass it to a method
	// that manages signal handlers.
	for ( int i = 0; i < _dnd_mgr.list_max; i++ ) {
		add_cb(get_row_at_index(i));
		}
	}

// Internal method that adds all important callbacks to a row.
// It also sets every row to be both source and target of a drag operation.
internal void add_cb (Widget row) {
	drag_source_set(row, BUTTON1_MASK, entries, DragAction.MOVE);
	drag_dest_set(row, DestDefaults.ALL, entries, DragAction.MOVE);
	row.drag_begin.connect(drag_begin_cb);
	row.button_press_event.connect(button_press_event_cb);
	row.drag_motion.connect(drag_motion_cb);
	row.drag_data_get.connect(drag_data_get_cb);
	row.drag_data_received.connect(drag_data_received_cb);
	row.drag_end.connect(drag_end_cb);
	}

// A final pointer drag handler that runs when drag ends.
void drag_end_cb (Widget row, DragContext c) {
	// This event fires when drag ends other way than via drag_data_received_cb.
	// Clears a visual state of the source by removing a css class marking
	// it as being dragged and inactive.
	_dnd_mgr.dragged(false);
	}

// A signal handler that fires when there was drop and some data
// has been transferred to a target.
void drag_data_received_cb (Widget row, DragContext c, int x, int y, SelectionData selection_data, uint info, uint time_) {
	// This event gets fired when the drop target receives data
	// (so drop happened).
	// First, remove a css class marking the source row as being dragged
	// and inactive.
	_dnd_mgr.dragged(false);
	// Then test whether the target and the source are equal - if so,
	// skip the data transfer
	if ( _dnd_mgr.source == (ListBoxRow)row ) return;
	// Grab the data and remove the source row from the list and insert it
	// at a new position.
	exchange_rows(_dnd_mgr.source_position, _dnd_mgr.target_position, ((Object[])selection_data.get_data ())[0]);
	}

// A signal handler that gets data from the source row.
void drag_data_get_cb (Widget row, DragContext c, SelectionData selection_data, uint info, uint time_) {
	// To enable data transfer during dnd operation use a SelectionData object.
	// In case of this widget it would be possible to use a property
	// of DndMgr object (which would be more straight forward), however the usual
	// method is left in place to enable cross widget communication.
	uchar[] row_obj = new uchar[( sizeof ( Object ))];
	var data = model.get_item(_dnd_mgr.source_position);
	((Object[])row_obj )[0] = data;
	selection_data.@set(Atom.intern_static_string (DATA_ID), 32, row_obj);
	}

// A signal handler that determines position of a pointer over the row
// it's called on.
bool drag_motion_cb (Widget row, DragContext c, int x, int y, uint time_) {
	// To save some cycles a number of times the update gets trigerred is limited.
	// This way instead of many times it happens just twice a row in each pass.
	// Test for equality of this row and a cached one is performed in a DndMgr
	// and blocks update if there is no need for it.
	// Then the DndMgr determines whether a half of the row has been crossed -
	// if so toggle css classes to show a bar that indicates possible next
	// placement of a drop.
	// Short explanation here - there are two ways (or more, but let's focus
	// on the following) to mark a place where a dropped row can be placed
	// (by marking I mean adding and removing css classes that add
	// a visual indicator).
	// One is to mark current or previous/next rows depending on a direction
	// in a list and position of cursor over the current row.
	// Another is to mark only the current row either atop or on its bottom
	// depending on cursor position. I prefer the former solution as
	// it greatly simplifies management of indication.
	_dnd_mgr.update((ListBoxRow)row, y);
	return false;
	}

void drag_begin_cb (Widget row, DragContext context) {
	// Initiating DndMgr's pointer driven drag.
	_dnd_mgr.pointer_drag((ListBoxRow)row);
	// Temporatily set a background filled css class so a Cairo surface
	// can get the image properly rendered.
	_dnd_mgr.drag_icon(true);
	// Grab widget's rectangle.
	var rect = Allocation();
	_dnd_mgr.source.get_allocation(out rect);
	// Create and paint the surface.
	Cairo.ImageSurface surface = new Cairo.ImageSurface (Cairo.Format.ARGB32, rect.width, rect.height);
	Cairo.Context ctx = new Cairo.Context (surface);
	_dnd_mgr.source.draw_to_cairo_context(ctx);
	// Shift the surface icon so it appears in correct relation to the pointer.
	surface.set_device_offset(-_dnd_mgr.start_x, -_dnd_mgr.start_y);
	// Display the surface.
	drag_set_icon_surface(context, surface);
	// Switch off the temporary class that was set above.
	_dnd_mgr.drag_icon(false);
	// Setting a css class that renders the widget in quasi-disabled manner
	// indicating its inactive state.
	// <notice> While it could be tempting to set state flags of this widget
	// to inactive it would make it unresponsive and logically detached
	// from the parent.
	// It's then better to have it responsive and only visually greyed
	// to retain the ability to properly catch events and count passes.
	// For instance, one of unwanted effects that occur when the row
	// has StateFlags.INSENSITIVE is a sudden jump of indicator when
	// a pointer passes over it.
	_dnd_mgr.dragged(true);
	}

bool button_press_event_cb (Widget row, EventButton event) {
	// For unknown reasons the row must be selected in button_press_event_cb
	// explicitly as after the drag'n'drop operation first pointer action doesn't
	// select the row (regardless of this event being actually fired).
	select_row((ListBoxRow)row);
	// Getting coordinates and storing them in _dnd_mgr, so in case a drag occurs
	// the widget will know the starting point of a pointer action.
	_dnd_mgr.start_x = (int) event.x;
	_dnd_mgr.start_y = (int) event.y;
	// Grab focus on the row so keyboard and pointer states remain consistent.
	get_selected_row().grab_focus();
	return false;
	}

void items_changed_cb (uint position, uint removed, uint added) {
	// Make sure the number of rows is updated and stored in the drag manager
	// so keyboard operations won't go too far.
	_dnd_mgr.list_max = model.get_n_items();
	if ( added > 0 ) {
		// Newly added rows must be bound to rows' specific signal handlers.
		for ( uint i = 0; i < added; i++ ) {
			add_cb(get_row_at_index((int)( i + position )));
			}
		// Select a row that was presumably added due to d'n'd action.
		// If it was added from outside no harm is inflicted as the below
		// condition makes sure the row number is lower than number of rows.
		if ( added == 1 && _dnd_mgr.target_position<_dnd_mgr.list_max ) {
			// Select and grab focus on newly added row so keyboard and pointer states
			// remain consistent
			var new_row = get_row_at_index(_dnd_mgr.target_position);
			select_row(new_row);
			new_row.grab_focus();
			}
		}
	}

// Mask ListBox methods that enable manual manipulation on rows and loading
// a model. Only locally managed ListStore based operations are allowed.
// NOT SURE IF I DID IT RIGHT - suggestions are welcome.
private new void add(){}
private new void remove() {}
private new void insert() {}
private new void set_selection_mode (SelectionMode m) {}
private new void bind_model (GLib.ListStore m, ListBoxCreateWidgetFunc f) {}

}
}

```
{% endtab %}

{% tab title="Клавиши" %}

{% endtab %}

{% tab title="ListBoxMrg" %}

{% endtab %}
{% endtabs %}

