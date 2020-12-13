# www-epub-tolino
Workflow to realize a one-click solution to 'push' interesting web sites to you tolino ebook reader in a wireless manner.

## Motivation

Reading long (more than ten minutes of reading time) articles on the computer
is something I really don't like. With the arival of my new ebook reader
(Tolino), I was wonding whether there is a intutive way to shift reading these
articles to the ebook reader universie (offline reading without disturbance on an 
eye-friendly e-ink screen). I was wishing for something like:

 1. Marking an intersting article in the web for reading on the ebook device.
 2. Automatically transfering this article to the ebook reader so that is
    magically available next time I am having the ebook reader at hand.

Within this document I want to summarize the workflow I am having running to realize this idea.


## Technical Concept

The general process I want to realize is:

 1. Mark intersting article during browsing (when lingering in web browser).
 2. Transform the web site to an epub file that can be read by the ebook reader.
 3. Uploading the epub file to the ebook reader.

After some investigations I came up with the following workflow that
implementes the process and realizes my inital idea.  
### Workflow

 1. Using [EpubPress](http://epub.press) browser plugin to transform the
    current (or selected) tabs to an epub file.
 2. Besides providing the epub for download, EpubPress can also send the file
    via email to a configured email addresse. I am using this feature.
 3. Adding a rule to my email backend that automatically sorts emails form
    EpubPress to a known folder on the mail storage side.
 4. On one of my headless devices at home (vacuum or router), I am now
    periodically polling this particula email storage (in my case it is an IMAP
folder)
 5. When a new mail from EpubPress is found, it is downloaded and the
    attachment is extracted.
 6. Finally the epub is uploaded via the [toloino cloud api CLI](https://github.com/hzulla/tolino-python)
    to your tolino cloud.


## Configuration

### Required tools

 * fetchmail (getting email from IMAP storage)
 * procmail (MDA)
 * mpack (extracting the epub attachments)

### E-Mail rules

Within my email provider configuration I setup a rule to sort all emails from
*noreply@epub.press* that contains an attachment to a folder named
*tolino-upload*.

### Polling for new epubs

I am downloading email from my outlook account with fetchmail.

```
poll outlook.office365.com with proto IMAP port 993
        user 'john'
        password 'doe'
        folder "tolino-upload"
        keep
	ssl
	sslfingerprint '32:77:16:07:55:27:05:16:C6:98:8F:B8:F4:3F:AB:6E'
```

See [fetchmail with office365](https://quornicus.wordpress.com/2016/01/08/fetchmail-office-365-configuration-rt/) to generate the ssl fingerprint.

### Extracting and storing the epubs

Each mail contains an attachment of the generated epub file. This shall be
extracted with procmail as mail delivery agent an mpack to a local folder.

The procmailrc file contains the following:
```
PATH=/usr/bin:/bin:/usr/local/bin:$HOME/bin:$PATH

:0 w
* ^From.*noreply@epub.press
| munpack -f -q -t -C $HOME/tolino-epubs

```

And the fetchmail configuration needs an additional line specifying an MDA:

```
mda procmail
```

Now just copy the configuration files to the proper places and create the output folder:
```
$ mkdir ~/tolino-epubs
$ cp fetchmailrc ~/.fetchmailrc
$ cp procmailrc ~/.procmailrc
```

Afterwards put your IMAP login creditials to a proper .netrc file (see netrc(5)).

### Uploading to the tolino cloud

### Running periodically
