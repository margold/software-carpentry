The cool thing about German banks is that they have all agreed on a single online-banking interface specification (called simply HBCI for Home Banking Computer Interface, or (in the newest version) FinTS for Financial Transaction Services). It's predictably being used by German online-banking and budgeting software, but the exciting thing is that it also makes it possible (if not necessarily easy) to integrate online banking into whatever existing life-maintenance pipelines you may have. Think sending yourself an sms when your total spending hits a threshold, or setting up a complex system of recurring payment triggers.

Neat, how can I start using it, you ask? As with many things in life, there's [a library that as far as I can tell is almost single-handedly written and maintained by a single German guy](http://www.aqbanking.de/) that can be used as-is, or through the included CLI tool. Let's see:

0. Adapt these instructions to your OS.

1. Install `aqbanking-cli` and some other tools (brew will link them to `/usr/local/bin/`). If brew doesn't install the newest version (the one shown with `brew info`), force building from source. 
  ```shell
  margold@home-macbook ~ $ brew install --build-from-source aqbanking
  ```

2. Make sure you have the versions you wanted.
  ```shell
  margold@home-macbook ~ $ aqbanking-cli versions
  Versions:
   AqBanking-CLI: 5.6.12
   Gwenhywfar   : 4.15.3.0
   AqBanking    : 5.6.12.0
  ```

3. Find the URL of your bank's FinTS server (google for "hbci" or "fints" and the name of your bank, or search for "hbci" or "fints" on your bank's website). I'm a comdirect customer and will be using https://fints.comdirect.de/fints.

4. Setup with your bank account. My bank (comdirect) works with tokentype `pin/tan` only, so that's what I'm going to use. 
  ```shell
  margold@home-macbook ~ $ aqhbci-tool4 adduser --tokentype=pintan \
  --context=1 \
  --bank=BLZ --user=ZUGANGSNUMMER \
  --server=SERVER \
  --hbciversion=300 \
  --username="VOLLER NAME"

  margold@home-macbook ~ $ aqhbci-tool4 adduserflags --bank=BLZ --user=ZUGANGSNUMMER -f forceSsl3
  ```

5. You might also need to setup `setitanmode`.
  ```shell
  margold@home-macbook ~ $ aqhbci-tool4 listitanmodes
  TAN Methods
  - 2900 (F900/V2/P2): TechnicalId (iTAN-Verfahren) [available]
  - 2901 (F901/V2/P2): TechnicalId901 (mobileTAN-Verfahren) [available]
  - 5902 (F902/V5/P2): MS1.0.0 (photoTAN-Verfahren) [not available]

  margold@home-macbook ~ $ aqhbci-tool4 setitanmode --bank=BLZ --user=ZUGANGSNUMMER -m 2900
  ```

6. Test whether everything works by fetching the `sysid`.
  ```shell
  margold@home-macbook ~ $ aqhbci-tool4 getsysid --customer=ZUGANGSNUMMER
  ```

7. Test some more.
  ```shell
  margold@home-macbook ~ $ aqhbci-tool4 getaccounts --user=ZUGANGSNUMMER  # get things first
  margold@home-macbook ~ $ aqhbci-tool4 listaccounts --verbose  # then display them
  ```

8. Create pin file for automatic requests.
  ```shell
  margold@home-macbook ~ $ aqhbci-tool4 mkpinlist > pinfile
  margold@home-macbook ~ $ vi pinfile  # add your pin
  margold@home-macbook ~ $ chmod 600 pinfile
  margold@home-macbook ~ $ # test with a command that requires a pinfile, such as
  margold@home-macbook ~ $ aqhbci-tool4 --pinfile=pinfile getaccounts --user=ZUGANGSNUMMER
  ```

9. You're all set now. Consult the CLI documentation for what you can do. To get you started, here's how you can [fetch a list of past transactions and export them to a csv file](https://gist.github.com/margold/b1993ac2225233844005673a4be66f93).


The tool is a bit fiddly, and it might take some sleuthing to figure out what additional flags you need to set for it to work with your bank. The [Handbuch](http://www.aquamaniac.de/sites/download/packages.php) provides many examples, and the mailing list (English-speaking, despite being mostly written in German) is fast and helpful. 
