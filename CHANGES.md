## 1.2.0

### Breakage

* Due to countless bugs we no longer include support for simple_form's type interrogation automatically. This allows using forms with non-AM objects. If you want full support for simple_form do as follows.

    ```ruby
    class SongForm < Reform::Form
      include ModelReflections
    ```

    Including this module will add `#column_for_attribute` and other methods need by form builders to automatically guess the type of a property.

### Changes

* `Form#save` with `Composition` now returns true only if all composite models saved.
* `Form::copy_validations_from` allows copying custom validators now.
* `::validates_uniqueness_of` now accepts options like `:scope`. Thanks to @cveneziani for a failing test and insight.
* `:skip_if`, `:skip_if: :all_blank`.

## 1.1.1

* Fix a bug where including a form module would mess up the options has of the validations (under older Rails).
* Fix `::properties` which modified the options hash while iterating properties.
* `Form#save` now returns the result of the `model.save` invocation.
* Fix: When overriding a reader method for a nested form for presentation (e.g. to provide an initial new record), this reader was used in `#update!`. The deserialize/update run now grabs the actual nested form instances directly from `fields`.
* `Errors#to_s` is now delegated to `messages.to_s`. This is used in `Trailblazer::Operation`.

## 1.1.0

* Deprecate first block argument in save. It's new signature is `save { |hash| }`. You already got the form instance when calling `form.save` so there's no need to pass it into the block.
* `#validate` does **not** touch any model anymore. Both single values and collections are written to the model after `#sync` or `#save`.
* Coercion now happens in `#validate`, only.
* You can now define forms in modules including `Reform::Form::Module` to improve reusability.
* Inheriting from forms and then overriding/extending properties with `:inherit` now works properly.
* You can now define methods in inline forms.
* Added `Form::ActiveModel::ModelValidations` to copy validations from model classes. Thanks to @cameron-martin for this fine addition.
* Forms can now also deserialize other formats, e.g. JSON. This allows them to be used as a contract for API endpoints and in Operations in Trailblazer.
* Composition forms no longer expose readers to the composition members. The composition is available via `Form#model`, members via `Form#model[:member_name]`.
* ActiveRecord support is now included correctly and passed on to nested forms.
* Undocumented/Experimental: Scalar forms. This is still WIP.

## 1.0.4

Reverting what I did in 1.0.3. Leave your code as it is. You may override a writers like `#title=` to sanitize or filter incoming data, as in

```ruby
def title=(v)
  super(v.strip)
end
```

This setter will only be called in `#validate`.

Readers still work the same, meaning that

```ruby
def title
  super.downcase
end
```

will result in lowercased title when rendering the form (and only then).

The reason for this confusion is that you don't blog enough about Reform. Only after introducing all those deprecation warnings, people started to contact me to ask what's going on. This gave me the feedback I needed to decide what's the best way for filtering incoming data.

## 1.0.3

* Systematically use `fields` when saving the form. This avoids calling presentational readers that might have been defined by the user.

## 1.0.2

* The following property names are reserved and will raise an exception: `[:model, :aliased_model, :fields, :mapper]`
* You get warned now when overriding accessors for your properties:

    ```ruby
    property :title

    def title
      super.upcase
    end
    ```

    This is because in Reform 1.1, those accessors will only be used when rendering the form, e.g. when doing `= @form.title`. If you override the accessors for presentation, only, you're fine. Add `presentation_accessors: true` to any property, the warnings will be suppressed and everything's gonna work. You may remove `presentation_accessors: true` in 1.1, but it won't affect the form.

    However, if you used to override `#title` or `#title=` to manipulate incoming data, this is no longer working in 1.1. The reason for this is to make Reform cleaner. You will get two options `:validate_processor` and `:sync_processor` in order to filter data when calling `#validate` and when syncing data back to the model with `#sync` or `#save`.

## 1.0.1

* Deprecated model readers for `Composition` and `ActiveModel`. Consider the following setup.
    ```ruby
      class RecordingForm < Reform::Form
        include Composition

        property :title, on: :song
      end
    ```

  Before, Reform would allow you to do `form.song` which returned the song model. You can still do this (but you shouldn't) with `form.model[:song]`.

  This allows having composed models and properties with the same name. Until 1.1, you have to use `skip_accessors: true` to advise Reform _not_ to create the deprecated accessor.

  Also deprecated is the alias accessor as found with `ActiveModel`.
    ```ruby
      class RecordingForm < Reform::Form
        include Composition
        include ActiveModel

        model :hit, on: :song
      end
    ```
  Here, an automatic reader `Form#hit` was created. This is deprecated as

  This is gonna be **removed in 1.1**.


## 1.0.0

* Removed `Form::DSL` in favour of `Form::Composition`.
* Simplified nested forms. You can now do
    ```ruby
    validates :songs, :length => {:minimum => 1}
    validates :hit, :presence => true
    ```
* Allow passing symbol hash keys into `#validate`.
* Unlimited nesting of forms, if you really want that.
* `save` gets called on all nested forms automatically, disable with `save: false`.
* Renaming with `as:` now works everywhere.
* Fixes to make `Composition` work everywhere.
* Extract setup and validate into `Contract`.
* Automatic population with `:populate_if_empty` in `#validate`.
* Remove `#from_hash` and `#to_hash`.
* Introduce `#sync` and make `#save` less smart.

## 0.2.7

* Last release supporting Representable 1.7.
* In ActiveModel/ActiveRecord: The model name is now correctly infered even if the name is something like `Song::Form`.

## 0.2.6

* Maintenance release cause I'm stupid.

## 0.2.5

* Allow proper form inheritance. When having `HitForm < SongForm < Reform::Form` the `HitForm` class will contain `SongForm`'s properties in addition to its own fields.
* `::model` is now inherited properly.
* Allow instantiation of nested form with emtpy nested properties.

## 0.2.4

* Accessors for properties (e.g. `title` and `title=`) can now be overridden in the form *and* call `super`. This is extremely helpful if you wanna do "manual coercion" since the accessors are invoked in `#validate`. Thanks to @cj for requesting this.
* Inline forms now know their class name from the property that defines them. This is needed for I18N where `ActiveModel` queries the class name to compute translation keys. If you're not happy with it, use `::model`.

## 0.2.3

* `#form_for` now properly recognizes a nested form when declared using `:form` (instead of an inline form).
* Multiparameter dates as they're constructed from the Rails date helper are now processed automatically. As soon as an incoming attribute name is `property_name(1i)` or the like, it's compiled into a Date. That happens in `MultiParameterAttributes`. If a component (year/month/day) is missing, the date is considered `nil`.

## 0.2.2

* Fix a bug where `form.save do .. end` would call `model.save` even though a block was given. This no longer happens, if there's a block to `#save`, you have to manually save data (ActiveRecord environment, only).
* `#validate` doesn't blow up anymore when input data is missing for a nested property or collection.
* Allow `form: SongForm` to specify an explicit form class instead of using an inline form for nested properties.

## 0.2.1

* `ActiveRecord::i18n_scope` now returns `activerecord`.
* `Form#save` now calls save on the model in `ActiveRecord` context.
* `Form#model` is public now.
* Introduce `:empty` to have empty fields that are accessible for validation and processing, only.
* Introduce `:virtual` for read-only fields the are like `:empty` but initially read from the decorated model.
* Fix uniqueness validation with `Composition` form.
* Move `setup` and `save` logic into respective representer classes. This might break your code in case you overwrite private reform classes.


## 0.2.0

* Added nested property and collection for `has_one` and `has_many` relationships. . Note that this currently works only 1-level deep.
* Renamed `Reform::Form::DSL` to `Reform::Form::Composition` and deprecated `DSL`.
* `require 'reform'` now automatically requires Rails stuff in a Rails environment. Mainly, this is the FormBuilder compatibility layer that is injected into `Form`. If you don't want that, only require 'reform/form'.
* Composition now totally optional
* `Form.new` now accepts one argument, only: the model/composition. If you want to create your own representer, inject it by overriding `Form#mapper`. Note that this won't create property accessors for you.
* `Form::ActiveModel` no longer creates accessors to your represented models, e.g. having `property :title, on: :song` doesn't allow `form.song` anymore. This is because the actual model and the form's state might differ, so please use `form.title` directly.

## 0.1.3

* Altered `reform/rails` to conditionally load `ActiveRecord` code and created `reform/active_record`.

## 0.1.2

* `Form#to_model` is now delegated to model.
* Coercion with virtus works.

## 0.1.1

* Added `reform/rails` that requires everything you need (even in other frameworks :).
* Added `Form::ActiveRecord` that gives you `validates_uniqueness_with`. Note that this is strongly coupled to your database, thou.
* Merged a lot of cleanups from sweet @parndt <3.

## 0.1.0

* Oh yeah.