# The only constant is an ID

I claim that all (user) data in a database should be mutable by default. Always. Well, *almost* all data. There are only four exceptions.

### **1\. Primary key**

A *dedicated* column that serves as the unique identifier of the record (ideally, a `GUID` or a consecutive number). *Every* table — without an exception, and I mean it — **must have** a *separate* column that uniquely identifies the record.

It may be tempting to use an existing column (such as `email` in a `user` table) as the primary key, but doing so makes it impossible to change a user’s email later on. Avoid this at all costs.

Instead, always use a separate column with a truly unique value and use this column exclusively to build relations and connect records in a database. That way, all other data (such as, for example, `email` or `slug`) can always be changed without breaking anything.

### **2\. Foreign keys**

Similar to primary keys, foreign keys take care of connecting records and maintaining relationships in a database. These must be always treated as read-only, as they ensure data integrity.

### **3\. Automatically generated values**

Many databases use computed values such as timestamps (e.g. `created_at`). All data that was generated without user input should generally be considered immutable. They usually provide meta information about a record and should be treated differently than user data.

### **4\. Records that are immutable by their nature**

Some database models include entirely read-only tables, where new data can be added, but existing data can never be modified. It can only be read. A common use case for this is a document activity log or a login history.

### **Conclusion**

All other (user-entered) data should be mutable — always. It may sound obvious, but it’s not always the case. Of course, there can be exceptions, but 99% of the time, this should be the default.

Suppose I’m using a SaaS where I can create a workspace with my company’s name and slug. In case I ever rebrand from ”Average company” to ”Superior company”, I want to be able to change the name, the slug, the admin user’s email address, and so on. If this is not possible, it’s usually a sign of weak database design (or poor UX).

Thanks for reading. Feel free to [follow me on Twitter](https://twitter.com/rbluethl).