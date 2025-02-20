//
//  ContentView.swift
//  open Placemat
//
//  Created by Bjorn on 2/19/25
//

import SwiftUI

// MARK: - Data Models

/// Represents an event with details like name, date, location, and attendees.
struct Event: Identifiable {
    let id = UUID()
    var name: String
    var date: Date
    var location: String
    var hostMaking: String
    var needsToBring: String
    var numberOfSeats: Int
    var attendees: [String]
}

/// Represents an invitation to an event with its status.
struct Invitation: Identifiable {
    let id = UUID()
    var event: Event
    var status: InvitationStatus
}

/// Status of an invitation.
enum InvitationStatus {
    case pending, accepted, declined
}

/// Represents a contact from the user's phone.
struct Contact: Identifiable {
    let id = UUID()
    var name: String
}

// MARK: - Main Content View

/// The main view with a tab-based interface.
struct ContentView: View {
    var body: some View {
        TabView {
            HomeView()
                .tabItem {
                    Image(systemName: "house")
                    Text("Home")
                }
            CalendarView()
                .tabItem {
                    Image(systemName: "calendar")
                    Text("Calendar")
                }
            EventsView() // Changed from GalleryView to EventsView
                .tabItem {
                    Image(systemName: "list.bullet")
                    Text("Events")
                }
            ContactsView()
                .tabItem {
                    Image(systemName: "person.2")
                    Text("Contacts")
                }
        }
        .accentColor(.red) // Red accent for branding
        .background(Color.white) // White background for branding
    }
}

// MARK: - Home View

/// Displays the user's pending invitations and events.
struct HomeView: View {
    @State private var events: [Event] = [
        Event(name: "Dinner Party", date: Date(), location: "My House", hostMaking: "Main Course", needsToBring: "Dessert", numberOfSeats: 6, attendees: ["Alice", "Bob"]),
        Event(name: "BBQ", date: Date().addingTimeInterval(86400), location: "Park", hostMaking: "Grilled Meats", needsToBring: "Sides", numberOfSeats: 10, attendees: ["Charlie"])
    ]
    @State private var invitations: [Invitation] = [
        Invitation(event: Event(name: "Birthday Bash", date: Date().addingTimeInterval(172800), location: "Friend's House", hostMaking: "Cake", needsToBring: "Drinks", numberOfSeats: 8, attendees: []), status: .pending)
    ]
    @State private var showingCreateEvent = false
    
    var body: some View {
        NavigationView {
            List {
                Section(header: Text("Pending Invitations").foregroundColor(.red)) {
                    ForEach(invitations.filter { $0.status == .pending }) { invitation in
                        HStack {
                            Text(invitation.event.name)
                            Spacer()
                            Button("Accept") { /* Accept logic */ }
                                .foregroundColor(.red)
                            Button("Decline") { /* Decline logic */ }
                                .foregroundColor(.red)
                        }
                    }
                }
                Section(header: Text("My Events").foregroundColor(.red)) {
                    ForEach(events) { event in
                        NavigationLink(destination: EventDetailView(event: event)) {
                            Text(event.name)
                        }
                    }
                }
            }
            .navigationTitle("Home")
            .foregroundColor(.black)
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button(action: { showingCreateEvent = true }) {
                        Image(systemName: "plus")
                            .foregroundColor(.red)
                    }
                }
            }
            .sheet(isPresented: $showingCreateEvent) {
                CreateEventView(events: $events)
            }
        }
    }
}

// MARK: - Create Event View

/// A form for creating a new event.
struct CreateEventView: View {
    @Binding var events: [Event]
    @State private var eventName = ""
    @State private var eventDate = Date()
    @State private var eventLocation = ""
    @State private var hostMaking = ""
    @State private var needsToBring = ""
    @State private var numberOfSeats = 0
    @Environment(\.presentationMode) var presentationMode
    
    var body: some View {
        NavigationView {
            Form {
                Section(header: Text("Event Details").foregroundColor(.red)) {
                    TextField("Event Name", text: $eventName)
                    DatePicker("Date and Time", selection: $eventDate)
                    TextField("Location", text: $eventLocation)
                }
                Section(header: Text("Host is Making").foregroundColor(.red)) {
                    TextField("What are you making?", text: $hostMaking)
                }
                Section(header: Text("Needs to Bring").foregroundColor(.red)) {
                    TextField("What needs to be brought?", text: $needsToBring)
                }
                Section(header: Text("Number of Seats").foregroundColor(.red)) {
                    Stepper(value: $numberOfSeats, in: 0...20) {
                        Text("Seats: \(numberOfSeats)")
                    }
                }
            }
            .navigationTitle("Create Event")
            .foregroundColor(.black)
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button("Save") {
                        let newEvent = Event(
                            name: eventName,
                            date: eventDate,
                            location: eventLocation,
                            hostMaking: hostMaking,
                            needsToBring: needsToBring,
                            numberOfSeats: numberOfSeats,
                            attendees: []
                        )
                        events.append(newEvent)
                        presentationMode.wrappedValue.dismiss()
                    }
                    .foregroundColor(.red)
                }
            }
        }
    }
}

// MARK: - Event Detail View

/// Displays details of an event and allows inviting contacts.
struct EventDetailView: View {
    let event: Event
    @State private var showingInviteView = false
    @State private var contacts: [Contact] = [
        Contact(name: "Alice"),
        Contact(name: "Bob"),
        Contact(name: "Charlie")
    ]
    
    var body: some View {
        Form {
            Section(header: Text("Event Details").foregroundColor(.red)) {
                Text("Name: \(event.name)")
                Text("Date: \(event.date, style: .date)")
                Text("Time: \(event.date, style: .time)")
                Text("Location: \(event.location)")
            }
            Section(header: Text("Host is Making").foregroundColor(.red)) {
                Text(event.hostMaking)
            }
            Section(header: Text("Needs to Bring").foregroundColor(.red)) {
                Text(event.needsToBring)
            }
            Section(header: Text("Available Seats").foregroundColor(.red)) {
                Text("\(event.numberOfSeats - event.attendees.count) / \(event.numberOfSeats)")
            }
            Section(header: Text("Attendees").foregroundColor(.red)) {
                ForEach(event.attendees, id: \.self) { attendee in
                    Text(attendee)
                }
            }
            Button("Invite People") {
                showingInviteView = true
            }
            .foregroundColor(.red)
        }
        .navigationTitle(event.name)
        .foregroundColor(.black)
        .sheet(isPresented: $showingInviteView) {
            InviteView(contacts: contacts, event: event)
        }
    }
}

// MARK: - Invite View

/// Allows selecting contacts to invite to an event.
struct InviteView: View {
    let contacts: [Contact]
    let event: Event
    
    var body: some View {
        List(contacts) { contact in
            HStack {
                Text(contact.name)
                Spacer()
                Image(systemName: "checkmark")
                    .foregroundColor(.red)
                    .opacity(0.5) // Placeholder for selection state
            }
        }
        .navigationTitle("Select Contacts")
        .foregroundColor(.black)
        .toolbar {
            ToolbarItem(placement: .navigationBarTrailing) {
                Button("Send Invites") {
                    print("Invites sent for \(event.name)")

​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​