I want to write about locale-aware number parsing and formatting, because when people write about locale-aware functions they usually only look at date/time, and yet here I am, needing to deal with things like "1.000,83" (that's German for `1000.83`).

The amount of information you need to parse "1.000,83" is pretty small: you need to know the symbols for `decimal_point` and `thousands_separator` for the current language-country combo. If you want to format numbers as monetary values, you will need a smattering of additional symbols: `currency_symbol` and the like. A list like that can be created once for every locale, maybe by the respective standardization authority, and used by implementations of any language that wants to provide locale-aware functions.

Instead, we have this.
- **Glibc**

  Comes with its own locale-specific data (https://sourceware.org/git/?p=glibc.git;a=tree;f=localedata/locales;h=1907f46a9f1d4686bff8e0c50fb702465df59040;hb=HEAD). Locale state is global, so in order to use several locales within the same app one has to switch.

- **CPython**

  Wraps C libraries (as it so often does!), in this case, [`locale.h`](https://github.com/python/cpython/blob/master/Modules/_localemodule.c). Hence, has the same problem as the C implementation. "On top of that, some implementations are broken in such a way that frequent locale changes may cause core dumps", Python3 documentation helpfully notes. It then goes on to say: 
  
  >It is generally a bad idea to call setlocale() in some library routine, since as a side effect it affects the entire program. If, when coding a module for general use, you need a locale independent version of an operation that is affected by the locale, you will have to find a way to do it without using the standard library routine. Even better is convincing yourself that using locale settings is okay.
  
  Thanks, Python3 documentation.
  
- **OpenJDK**

  Comes with its own locale-specific data (http://hg.openjdk.java.net/jdk8/jdk8/jdk/file/687fd7c7986d/src/share/classes/sun/util/cldr/resources/21_0_1/common/main/de.xml#l2631) that can be used from the `NumberFormatter` class.
  
- **Ruby**

  I googled for half and hour and I still can't figure it out. I'm sure [6-year old articles like this one](http://blog.makandra.com/2010/03/how-to-use-the-comma-as-decimal-separator-in-activerecord-columns-and-text-fields/) are not indicative of the current state.
  
- **Elixir/Erlang**

  As far as I can tell, doesn't come with i18n features in the standard libraries. There is some stuff happening in hex, but nothing definitive or well-used so far.

There *is* one positive side-effect to everybody rolling their own. Reading the commit histories of locale-related libraries is like watching Völkerverständigung in action. When the Communist leaders of what would become Russia proclaimed "Workers of the world, unite!", nowhere would this slogan ring more true than in a library like [Arrow](https://github.com/crsmithdev/arrow/commits/master/arrow/locales.py), with its multilingual grammatical wisdom and general good cheer in the commit messages.
