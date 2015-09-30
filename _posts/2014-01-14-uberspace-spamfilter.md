---
layout: 	      post
title:  	      "uberspace spamfilter"
banner_image:   jet.jpg
meta:		        uberspace, mail, spamfilter, dspam
---

hier meine erfahrungen bzw. ideen zu meinem spamfilter bei [uberspace](uberspace.de).

meine idee basiert im grunde auf dem [dspam beispiel](http://uberspace.de/dokuwiki/mail:dspam) von uberspace, welches ich ein wenig angepasst habe und so z.b. [spamassassin](http://uberspace.de/dokuwiki/mail:spamassassin) mit integriert habe.

es wird eine imap ordnerstruktur angelegt, in der nacher die spammails landen und die verwendet wird, um false positives/negatives zu markieren.

{% highlight text %}
\
 \___ Inbox
  \
   \___ Spamfilter
         \
          \___ als Ham lernen
           \
            \___ als Spam erkannt
             \
              \___ als Spam lernen
{% endhighlight %}

zasätzlich dazu kann man noch zwei weitere ordner anlegen, welche dann zum training historischer mails verwendet werden können. dazu später mehr.

zuerst gibt es eine kleine abwandlung der uberspace skripts, welches ich als `~/.mailfilter-dspam` gesichert habe.

{% highlight bash %}
# where to put mails
MAILDIR = "$HOME/Maildir"

import EXT
if ( $EXT )
{
  CHECKMAILDIR = `dumpvuser $EXT | grep '^Directory' | awk '{ print $2 }'`
  if ( $CHECKMAILDIR )
  {
    MAILDIR=$CHECKMAILDIR
  }
}
 
# rules that should be applied before DSPAM go here

# Aufruf SpamAssassin
xfilter "/usr/bin/spamc" 
 
# now show the mail to DSPAM
xfilter "/package/host/localhost/dspam/bin/dspam --mode=teft --deliver=innocent,spam --stdout"
 
# check wether our folder structure is in place, create it if not
`test -d "$MAILDIR/.Spamfilter"`
if( $RETURNCODE == 1 )
{
  `maildirmake "$MAILDIR/.Spamfilter"`
  `test -d "$MAILDIR/.Spamfilter.als Spam erkannt"`
  if( $RETURNCODE == 1 )
  {
    `maildirmake "$MAILDIR/.Spamfilter.als Spam erkannt"`
  }
  `test -d "$MAILDIR/.Spamfilter.als Spam lernen"`
  if( $RETURNCODE == 1 )
  {
    `maildirmake "$MAILDIR/.Spamfilter.als Spam lernen"`
  }
  `test -d "$MAILDIR/.Spamfilter.als Ham lernen"`
  if( $RETURNCODE == 1 )
  {
    `maildirmake "$MAILDIR/.Spamfilter.als Ham lernen"`
  }
}
 
# put away what DSPAM considers to be SPAM
if( /^X-DSPAM-Result: Spam/ )
{
  to "$MAILDIR/.Spamfilter.als Spam erkannt/"
}
 
# further rules go here
{% endhighlight %}

was dieses skript macht ist folgendes:

* es ruft spamassassin auf, aber nur um header zu schreiben (die kann dspam dann später auswerten)
* es ruft dspam auf um header zu schreiben
* es erzeugt bei bedarf die oben genannte ordnerstruktur
* falls dspam etwas als spam klassifiziert wird es nach `als Spam erkannt` verschoben

nicht vergessen: `chmod 0600 .mailfilter-dspam`

das muss jetzt mittels `maildrop` aufgerufen werden. dazu kann z.b. meine qmail datei `.qmail-christian` wie folgt aussehen:

{% highlight bash %}
|maildrop $HOME/.mailfilter-dspam
{% endhighlight %}

nun muss es aber auch noch einen weg geben, den mailfilter anzulernen. dazu habe ich das skript `~/bin/dspam-learn`, wieder eine abwandlung von uberspace.

{% highlight bash %}
#!/bin/bash
########################################################################
# 2013-07-10 Christopher Hirschmann c.hirschmann@jonaspasche.com
########################################################################
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <http://www.gnu.org/licenses/>.
#
########################################################################

while getopts ":hv" Option
do
    case $Option in
        h       )       echo -e "Usage:\n-v\tbe verbose\n-h\tthis help message"; exit 0;;
        v       )       VERBOSE=1; ;;
        *       )       echo -e "ERROR: Unimplemented option chosen: -${OPTARG}"; exit 1;;
    esac
done

for DIR in $HOME/Maildir `find ${HOME}/users/ -mindepth 1 -maxdepth 1 -type d`;
do
    if [ "${VERBOSE}" == "1" ];
    then
        echo "Looking for mails in ${DIR}."
    fi
    if [ -d "$DIR/.Spamfilter.als Spam lernen" ];
    then
        for SUBDIR in new cur ;
        do
            IFSSAVE=$IFS;
            while IFS= read -d $'\0' -r file;
            do
                if [ "${VERBOSE}" == "1" ];
                then
                    echo "Eating \"$file\". Yuk!";
                fi
                nice -n 19 ionice -c3 /package/host/localhost/dspam/bin/dspam --class=spam --source=error --stdout < "${file}";
		echo "mv \"$file\" \"$DIR/.Spamfilter.als Spam erkannt/$SUBDIR/\""
		mv "$file" "$DIR/.Spamfilter.als Spam erkannt/$SUBDIR/"
            done < <(find "$DIR/.Spamfilter.als Spam lernen/$SUBDIR" -type f -print0)
            IFS=$IFSSAVE;
        done
    fi
    if [ -d "$DIR/.Spamfilter.Spam neu" ];
    then
        for SUBDIR in new cur ;
        do
            IFSSAVE=$IFS;
            while IFS= read -d $'\0' -r file;
            do
                if [ "${VERBOSE}" == "1" ];
                then
                    echo "Eating \"$file\". Yuk!";
                fi
                nice -n 19 ionice -c3 /package/host/localhost/dspam/bin/dspam --class=spam --source=inoculation --stdout < "${file}";
                rm -f "${file}";
            done < <(find "$DIR/.Spamfilter.Spam neu/$SUBDIR" -type f -print0)
            IFS=$IFSSAVE;
        done
    fi
    if [ -d "$DIR/.Spamfilter.als Ham lernen" ];
    then
        for SUBDIR in new cur ;
        do
            IFSSAVE=$IFS;
            while IFS= read -d $'\0' -r file;
            do
                if [ "${VERBOSE}" == "1" ];
                then
                    echo "Eating \"$file\". Yum!";
                fi
                nice -n 19 ionice -c3 /package/host/localhost/dspam/bin/dspam --class=innocent --source=error --stdout < "${file}";
                #rm -f "${file}";
				echo "mv \"$file\" \"$DIR/$SUBDIR/\""
				mv "$file" "$DIR/$SUBDIR/"
            done < <(find "$DIR/.Spamfilter.als Ham lernen/$SUBDIR" -type f -print0)
            IFS=$IFSSAVE;
        done
    fi
    if [ -d "$DIR/.Spamfilter.Ham neu" ];
    then
        for SUBDIR in new cur ;
        do
            IFSSAVE=$IFS;
            while IFS= read -d $'\0' -r file;
            do
                if [ "${VERBOSE}" == "1" ];
                then
                    echo "Eating \"$file\". Yum!";
                fi
                nice -n 19 ionice -c3 /package/host/localhost/dspam/bin/dspam --class=innocent --source=corpus --stdout < "${file}";
                rm -f "${file}";
            done < <(find "$DIR/.Spamfilter.Ham neu/$SUBDIR" -type f -print0)
            IFS=$IFSSAVE;
        done
    fi
done

if [ "${VERBOSE}" == "1" ];
then
    echo "Done looking for mails."
fi
{% endhighlight %}

nicht vergessen: `chmod +x ~/bin/dspam-learn`

was es macht ist folgendes

* mails in `als Ham lernen` welden als *ham* trainiert und danch in die inbox geschoben
* mails in `als Spam lernen` welden als *spam* trainiert und danch in *als Spam erkannt* geschoben
* ***achtung, das klappt hier nur mit mails die dpam schon gefiltert hat!***
* fall jemand den ordner `Spam neu` oder `Ham neu` angelegt hat (wird nicht automatisch angelegt!), werden alte mails, also mails die dspam vorher noch nicht gesehen hat, eingelernt. diese mails werden dann gelöscht, d.h. man sollte hier nur *kopien* ablegen!

wenn das alles klappt muss man noch einen service anlegen. das kann in etwa so passieren:

{% highlight bash %}
test -d ~/service || uberspace-setup-svscan
runwhen-conf ~/etc/dspam-learn "~/bin/dspam-learn"
sed -i -e "s/^RUNWHEN=.*/RUNWHEN=\",M=`awk 'BEGIN { srand(); printf("%d\n",rand()*60) }'`\"/" ~/etc/dspam-learn/run
ln -s ~/etc/dspam-learn ~/service
{% endhighlight %}

damit sollte das einlernen einmal in der stunde passieren.

jetzt sollten wir noch unsere db fit halten. z.b. so:

{% highlight bash %}
test -d ~/service || uberspace-setup-svscan
runwhen-conf ~/etc/dspam_clean_hashdb "/usr/local/bin/dspam_clean_hashdb"
sed -i -e "s/^RUNWHEN=.*/RUNWHEN=\",H=`awk 'BEGIN { srand(); printf("%d\n",rand()*24) }'`\"/" ~/etc/dspam_clean_hashdb/run
ln -s ~/etc/dspam_clean_hashdb ~/service
{% endhighlight %}

details was hier passiert findet man aber in der uberspace doku.

so das ist alles, so funktioniert für mich mein spamfilter. klappt sehr gut nach einer einlernphase. 

was ich für mich noch gemacht habe ist einen alten spamcorpus einzulernen. das ist natürlich nicht ideal, aber bietet zumindest eine basis. ich habe dafür diesen [corpus](http://spamassassin.apache.org/publiccorpus/) verwendet. das archiv kann im filesystem der mailbox dann direkt unter `Spam neu` entpackt werden.
