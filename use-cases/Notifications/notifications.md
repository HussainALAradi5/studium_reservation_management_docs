# Notification use cases

### UC-NTF-01 Staff hear about a new booking
- **Actor:** System → stadium manager (or every admin when the stadium has no manager)
- **Main flow:** Customer books → after the booking commits, a `reservation.requested` notification is stored → the manager's bell badge updates on the next poll (≤ 60 s) with a toast.
- **Result:** Opening it marks it read and goes to the booking.

### UC-NTF-02 Customer follows their booking
- **Actor:** Customer
- **Main flow:** Confirm, reject (with reason), cancel by staff (with reason), reschedule by staff, complete, payment, refund, and expiry each send one notification to the customer.
- **Alternate flow:** The customer's own actions are not echoed back to them; the other party is told instead.

### UC-NTF-03 Read and clear notifications
- **Actor:** Any signed-in user
- **Main flow:** Bell → latest 8 → open one (marked read) or "Mark all read" → inbox page with unread / type filters and pages of 15.
- **Failure flows:** Opening another user's notification code → 404.
