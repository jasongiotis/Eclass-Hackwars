## Open eClass 2.3

Το repository αυτό περιέχει μια __παλιά και μη ασφαλή__ έκδοση του eclass.
Προορίζεται για χρήση στα πλαίσια του μαθήματος
[Προστασία & Ασφάλεια Υπολογιστικών Συστημάτων (ΥΣ13)](https://ys13.chatzi.org/), __μην τη
χρησιμοποιήσετε για κάνενα άλλο σκοπό__.


### Χρήση μέσω docker
```
# create and start (the first run takes time to build the image)
docker-compose up -d

# stop/restart
docker-compose stop
docker-compose start

# stop and remove
docker-compose down -v
```

To site είναι διαθέσιμο στο http://localhost:8001/. Την πρώτη φορά θα πρέπει να τρέξετε τον οδηγό εγκατάστασης.


### Ρυθμίσεις eclass

Στο οδηγό εγκατάστασης του eclass, χρησιμοποιήστε __οπωσδήποτε__ τις παρακάτω ρυθμίσεις:

- Ρυθμίσεις της MySQL
  - Εξυπηρέτης Βάσης Δεδομένων: `db`
  - Όνομα Χρήστη για τη Βάση Δεδομένων: `root`
  - Συνθηματικό για τη Βάση Δεδομένων: `1234`
- Ρυθμίσεις συστήματος
  - URL του Open eClass : `http://localhost:8001/` (προσοχή στο τελικό `/`)
  - Όνομα Χρήστη του Διαχειριστή : `drunkadmin`

Αν κάνετε κάποιο λάθος στις ρυθμίσεις, ή για οποιοδήποτε λόγο θέλετε να ρυθμίσετε
το openeclass από την αρχή, διαγράψτε το directory, `openeclass/config` και ο
οδηγός εγκατάστασης θα τρέξει ξανά.

## 2023 Project 1

Εκφώνηση: https://ys13.chatzi.org/assets/projects/project1.pdf


### Μέλη ομάδας

- 1900041, Ιάσονας Γιώτης
- 1900070, Άγγελος Καμάρης

### Report

### Πρώτο Κομμάτι

- SQL INJECTIONS:

  Στο αρχικό site για την προστασία από sql injections, υπάρχει το φίλτρο ``` mysql_real_escape_string ```. Δεν καταφέραμε να βρούμε τρόπο να σπάσουμε αυτήν την άμυνα καθώς η ίδια η 
sql στην αρχή χρησιμοποιεί ``` mysql_query('SET NAMES utf8'); ``` κάτι το οποίο την καθιστά ασφαλή καθώς δεν έχει ευάλωτους χαρακτήρες, (όπως θα είχε η gbk π.χ). Παρόλα αυτά υπάρχουν σελίδες που τα inputs δεν φιλτράρονται. Μερικά παραδείγματα είναι κατά την δημιουργία και την επεξεργασία στοιχείων μαθητή, ενώ τα πεδία username και password, φιλτράρονται, τα υπόλοιπα (email, name, prename, am) δεν φιλτράρονται , (στο am δεν ελέγχεται καν αν θα μπει αριθμός). Με αυτόν τον τρόπο κάποιος μπορεί να βάλει: ``` ', prenom = (SELECT pass FROM (SELECT password as pass FROM user WHERE user_id = 1) AS p), --  ```, στο επώνυμο, και να αποθηκευτεί έτσι στο όνομα ο κωδικός του admin. Αντιστοίχως ευάλωτα είναι και τα περισσότερα πεδία στην περιοχή συζητήσεων, καθώς αν στα url των σελιδων viewtopic, newtopic και reply, καθώς βάζοντας sql injections στην μεταβλητή topic ή forum, για το newtopic, θα εμφανιστεί στα breadcrumps, ή στον τίτλο, το hash του κωδικού του admin. (π.χ. το: ``` topic=1)and 1=0 union select 1,password from eclass.user --   ```[το topic δεν τσεκάρει αν εισάγετε αριθμός]), στο viewtopic, μας δίνει τον κωδικό του admin. Στο ανέβασμα αρχείων, ένα αρχείο με το όνομα: ``` ', (SELECT password FROM eclass.user WHERE user_id = '1')); -- '``` , θα πάρουμε το hash του admin, στα σχόλια της εργασίας. Τέλος κατά την απεγγραφή από το μάθημα  υπάρχει η μεταβλητή cid στο url, η οποία μπορεί να μας γυρίσει το hash του admin. Υπάρχουν και επιθέσεις σε σελίδες που αφορούν τον admin, σε αυτές θα αναφερθούμε στα csrf.

  Για την καταπολέμηση των από πάνω προβλημάτων, έβαλα prepared statements σε όσες περισσότερες σελίδες μπορούσα, ασχέτως αν είχαν ήδη φίλτρο ή όχι, χρησιμοποιόντας ``` mysqli ```, σε σελίδες που μπορούσε να χρησιμοποιηθεί, αλλιώς έβαζα φίλτρο στις υπόλοιπες (ακόμα και σε σελίδες που δεν μπορούσα να κάνω επίθεση). Άλλο ένα λάθος που βρήκα ήταν ότι για να τρέξει η sql, καλούνταν η συνάρτηση db_query, η οποία σε περίπτωση λάθους εκτύπωνε το αποτέλεσμα του query, κάτι το οποίο δεν επιτρέπω. Τέλος σε μεταβλητές που εισάγωνται ως αριθμητικό ποσό, αλλά δεν ελέγχονται, βάζω να εισάγεται string, ώστε με το φίλτρο να καταπολεμάται το πρόβλημα των επιθέσεων.


- CSRF ATTACKS:
 
  Η ιστοσελίδα που μας δόθηκε, δεν είχε σε κανένα κομμάτι προστασία από csrf attacks. Όχι μόνο αυτό αλλά δεν έκανε και την καλύτερη χρήση post και get, για την αποφυγή τέτοιων επιθέσεων (π.χ. η διαγραφή χρήστη γίνεται με get, όπου στο url απλά προσθέτεται μια ένδειξη για την διαγραφη και το id του χρήστη). Αυτό είχε σαν αποτέλεσμα να μπορούν να γίνουν πολλές επιθέσεις csrf, ενδεικτικά θα αναφέρω μερικές που βρήκα εγώ επικίνδυνες: Διαγραφή χρήστη, Χρήση Admin, δημιουργία μαθήματος, δημιουργία forum και topic, αλλαγή των δεδομένων της βάσης, αλλαγή στοιχείων χρήστη, απεγγραφή χρήστη από το μάθημα, αλλαγή κωδικού χρήστη, κατέβασμα αρχείων και άλλα παρόμοια με αυτά. Πέρα όμως από αυτά τα προβλήματα υπήρχαν και 2 περιπτώσεις που το csrf, σε συνδιασμό με sqli injections, μπορούσε να εμφανίσει τον κωδικό του admin, στις σελίδες forum_admin και list_users. Στο πρώτο, βάζοντας στον Όνομα περιοχής συζητήσεων: ``` aaa' , (SELECT password FROM eclass.user WHERE user_id='1'), '2', '1', '5', '');--  ```, θα εμφανιστεί στα σχόλια αυτής της περιοχής το hash του κωδικού του admin, ενώ στο δεύτερο, από την σελίδα /modules/admin/search_user.php, αν κάποιος βάλει το: ``` ' union select password, password, password, password, password, password from eclass.user where user_id=1--  ```, στο επώνυμο ή πατηθεί το link: ``` http://localhost:8001/modules/admin/listusers.php?user_surname=%27%20%20union%20select%20password,%20password,password,password,password,password%20from%20eclass.user%20where%20user_id=1--+&search_submit=%CE%91%CE%BD%CE%B1%CE%B6%CE%AE%CF%84%CE%B7%CF%83%CE%B7 ```, θα φανούν στην σελίδα σε όλα τα πεδία αναζήτησης το hash του κωδικού του admin. Πέρα από αυτά σε αυτές τις σελίδες αλλά και σε άλλες με παρόμοιο τρόπο μπορούν να γίνουν και xss attacks.
  
  Για την καταπολέμηση αυτού του θέματος, δημιούργησα το αρχείο:  ``` openeclass\include\csrf_func.php ```, το οποίο και κάνω include στο αρχείο ```openeclass\include\baseTheme.php ```, στην γραμμή 51, ώστε να είναι σε όλες μου τις σελίδες. Αυτό το αρχείο έχει 2 συναρτήσεις, την: makeToken(), η οποία ελέγχει αν υπάρχει ήδη token και αν δεν υπάρχει το δημιουργεί και το αποθηκεύει στο SESSION, και την:  checkToken(): η οποία ελέγχει αν υπάρχει token στο post ή στο request και είναι ίδιο με αυτό στο session αλλιώς σβήνει το token που υπάρχει αυτή την στιγμή στο session (σε περίπτωση που κάποιος το αντιγράψει) και εμφανίζει μύνημα λάθους στον χρήστη.
Αυτές οι 2 συναρτήσεις χρησιμοποιούνται στις φόρμες αλλά και σε μερικά urls, έτσι ώστε να βεβαιωθούμε ότι ο χρήστης που μπαίνει σε μια σελίδα, δεν μπαίνει με redirect. Αυτό το επιτυγχάνουμε στις φόρμες προσθέτοντας το token σαν hidden input, ενώ στα urls πρόσθεσα μια μεταβλητή σε αυτά που έχει το token. Τέλος κάτι επιπλέον που έκανα, σε περίπτωση που κάποιος αποπειραθεί να αντιγράψει το session id, έβαλα στο ``` openeclass\index.php ``` την παράμετρο στο cookie, να μην μπορεί να εμφανιστεί το document.cookie.
  
  
  
- XSS ATTACKS :
Γενικά στα περισσότερα σημεία που υπήρχε user input έπρεπε εφαρμοστούν φίλτρα διότι υπήρχαν αρκετά vulerabilities π.χ αν ο χρήστης διάλεγε να κάνει edit το profile του  και έβαζε στο surname <script> alert(1) </script> εμφανιζόταν μύνημα έπειτα απο την αλλαγή στοιχείων . Επίσης xss attacks γινόντουσαν και απο links  όπως :

	http://localhost:8001/modules/profile/profile.php/"><script>alert(1)</script>

	http://localhost:8001/modules/work/work.php?id='--<script>alert(1)</script>

	http://localhost:8001/index.php/%27"/><script>alert(1)</script>

	http://localhost:8001/modules/phpbb/reply.php?topic=2&forum=1&message=<script>alert(1)</script>&submit=Yποβολή

Και άλλα πολλά σε σχεδόν όποιο σημείο είχε user input που άλλαζε απο link . Αρκετά xss attacks είναι αρκετά παρόμοια και με αυτά που αναφέρονται στο csrf section

Για την ασφάλιση της ιστοσελίδας στον τομέα αυτό χρησημοποιήθηκε η συνάρτηση htmlentities που απαλοίφει τους χαρακτήρες
που μπορούν να χρησιμοποιηθούν για να εκτελέσουν κάποιο scrit στην σελίδα. Κάποια κομμάτια που χρειαζόντουσαν αρκετό patchaρισμα για xss attacks ηταν : profile.php , viewforum.php , work.php ,forum_admin.php ,Basetheme.php ,MessageList.php και άλλα . 

Πριν απο αυτές τις αλλαγές υπήρχαν μεταξύ άλλων κάποια απο τα παρακάτω attacks :

- O χρήστης έκανε edit το profile του και έβαζα Nom ή Prenom <scirpt> alert(1) </script> 
- O χρήστης ανέβαζε εργασία και στην περιγραφή της έβαζε  <scirpt> alert(1) </script> 
- Ο χρήστης έστλενε μύνημα στην τηλεργασία  <scirpt> alert(1) </script> 
- Ο χρήστης μέσω της ανταλλαγής αρχείων έστελνε αρχείο στον admin και στην περιγραφή έβαζε <scirpt> alert(1) </script> 


- File injections : 
Τα 3 βασικά μέρη που έκαναν το site ευάλωτο σε file injections ήταν : Oι εργασίες , Η ανταλαγή αρχείων , Τα μη προστατευμένα links του server . 
Το site ήταν ευάλωτο στο εξής attack :
 Ανέβασμα εργασίας ή αρχείου για ανταλλαγή 
 Eύρεση path του αρχείου κάνοντας προσπέλαση σε Link όπως http://localhost:8001/include/ το οποίο μετέφερε τον χρήστη σε σελίδα που φενόταν το filesystem του server . 
 Εκτέλεση του αρχείου απο το path.
 Το αρχείο ήταν ένα php αρχείο το οποίο είτε έκανε unlink το root με την χρήση της εντολής unlink είτε διάβαζε τα αρχεία ρυθμίσεων (γνωρίζοντας το path) βλέποντας έτσι τον κωδικό της βάσης δεδομένων 

 Για να προστατευτόυμε απο τέτοιου είδους attacks έγιναν οι εξής αλλαγές .
 Τα αρχεία αποθηκεύονται στο filesystem σε zips και δίονται για download σε μορφή zip με την χρήση της προεγκατεστημένης στο vanilla eclass βιβλιοθήκης PCLZIP . (work.php ,dropbox_submit.php ,dropbox_download.php)

 Στον φάκελο cong στο 000-default.conf προσθέθηκε ο εξής κανόνας ο οποίος απαγορεύει την προβολή σελιδών του site χωρίς Index

 	<Directory /var/www/openeclass>

        Options -Indexes +FollowSymLinks

        AllowOverride None

        Require all granted

	</Directory>

  

Συμπληρώστε εδώ __ένα report__ που
- Να εξηγεί τι είδους αλλαγές κάνατε στον κώδικα για να προστατέψετε το site σας (από την κάθε επίθεση).
- Να εξηγεί τι είδους επιθέσεις δοκιμάσατε στο αντίπαλο site και αν αυτές πέτυχαν.
