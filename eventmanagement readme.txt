Frontend (Angular) Implementation:
Install Angular CLI and Generate Project:
bash
Copy code
ng new event-management
cd event-management
ng generate component event-list
ng generate component event-details
ng generate component event-create
ng generate service event
Service for Handling API Calls:
typescript
Copy code
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class EventService {

  private baseUrl = 'http://localhost:8080/event';

  constructor(private http: HttpClient) { }

  getAllEvents(): Observable<any> {
    return this.http.get(`${this.baseUrl}/events`);
  }

  getEventById(eventId: number): Observable<any> {
    return this.http.get(`${this.baseUrl}/${eventId}`);
  }

  getAvailableEvents(): Observable<any> {
    return this.http.get(`${this.baseUrl}/available`);
  }

  getEventsBySpeaker(speaker: string): Observable<any> {
    return this.http.get(`${this.baseUrl}/speaker/${speaker}`);
  }

  createEvent(event: any): Observable<any> {
    return this.http.post(`${this.baseUrl}`, event);
  }

  updateEvent(event: any): Observable<any> {
    return this.http.put(`${this.baseUrl}`, event);
  }

  bookSeats(eventId: number, numOfSeats: number): Observable<any> {
    return this.http.put(`${this.baseUrl}/book/${eventId}/${numOfSeats}`, null);
  }

  deleteEvent(eventId: number): Observable<any> {
    return this.http.delete(`${this.baseUrl}/${eventId}`);
  }
}
Component to List All Events:
typescript
Copy code
import { Component, OnInit } from '@angular/core';
import { EventService } from '../event.service';

@Component({
  selector: 'app-event-list',
  templateUrl: './event-list.component.html',
  styleUrls: ['./event-list.component.css']
})
export class EventListComponent implements OnInit {

  events: any;

  constructor(private eventService: EventService) { }

  ngOnInit(): void {
    this.loadEvents();
  }

  loadEvents(): void {
    this.eventService.getAllEvents().subscribe(data => {
      this.events = data;
    });
  }

  deleteEvent(eventId: number): void {
    this.eventService.deleteEvent(eventId).subscribe(() => {
      this.loadEvents(); // Reload events after deletion
    });
  }
}
Event List HTML (event-list.component.html):
html
Copy code
<div *ngFor="let event of events">
  <p>{{ event.name }} | {{ event.speaker }} | Seats Booked: {{ event.seatsBooked }}</p>
  <button (click)="deleteEvent(event.eventId)">Delete Event</button>
</div>
Running the Project:
Start the Spring Boot backend:
bash
Copy code
mvn spring-boot:run
Start the Angular frontend:
bash
Copy code
ng serve
The frontend will connect to the backend through the provided REST APIs. This will allow users to manage events (CRUD operations), book seats, and filter events with available seats.







You said:
give json body for post api
ChatGPT said:
ChatGPT
The POST API for creating a new event expects a JSON body with the event details (except for eventId, which will be auto-generated). Here’s an example JSON body for the /event POST API:

Example JSON Body for POST Request:
json
Copy code
{
  "name": "Tech Conference 2024",
  "time": "2024-11-10T10:00:00",
  "venue": "Conference Hall A, Tech Park",
  "speaker": "John Doe",
  "seatsBooked": 0
}
Explanation:
name: The name of the event.
time: The date and time of the event in ISO 8601 format (yyyy-MM-ddTHH:mm:ss).
venue: The location where the event will take place.
speaker: The speaker for the event.
seatsBooked: This field will typically be set to 0 when creating a new event since no seats are booked initially.
Note: When you send this request, the backend will auto-generate an eventId and store it in the database. The response will include the full event details, including the eventId.






You said:
give unit tests for this project
ChatGPT said:
ChatGPT
For writing unit tests in Spring Boot, we generally use JUnit 5 (JUnit Jupiter) along with Mockito to mock dependencies. Below are examples of unit tests for the service layer and the controller layer in the project.

1. Unit Tests for Service Layer (EventService)
We'll mock the EventRepository and test the business logic of the EventService.

Maven Dependency for Testing:
Add these dependencies in your pom.xml:

xml
Copy code
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <scope>test</scope>
</dependency>
EventServiceTest.java:
java
Copy code
package com.example.eventmanagement.service;

import com.example.eventmanagement.entity.Event;
import com.example.eventmanagement.repository.EventRepository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;

import java.util.Arrays;
import java.util.List;
import java.util.Optional;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

class EventServiceTest {

    @Mock
    private EventRepository eventRepository;

    @InjectMocks
    private EventService eventService;

    private Event event1;
    private Event event2;

    @BeforeEach
    void setUp() {
        MockitoAnnotations.openMocks(this);

        event1 = new Event();
        event1.setEventId(1L);
        event1.setName("Tech Conference 2024");
        event1.setSpeaker("John Doe");
        event1.setSeatsBooked(50);

        event2 = new Event();
        event2.setEventId(2L);
        event2.setName("AI Summit 2024");
        event2.setSpeaker("Jane Smith");
        event2.setSeatsBooked(10);
    }

    @Test
    void getAllEvents() {
        when(eventRepository.findAll()).thenReturn(Arrays.asList(event1, event2));

        List<Event> events = eventService.getAllEvents();

        assertEquals(2, events.size());
        verify(eventRepository, times(1)).findAll();
    }

    @Test
    void getEventById() {
        when(eventRepository.findById(1L)).thenReturn(Optional.of(event1));

        Event event = eventService.getEventById(1L);

        assertNotNull(event);
        assertEquals("Tech Conference 2024", event.getName());
        verify(eventRepository, times(1)).findById(1L);
    }

    @Test
    void getAvailableEvents() {
        when(eventRepository.findBySeatsBookedLessThan(100)).thenReturn(Arrays.asList(event1, event2));

        List<Event> availableEvents = eventService.getAvailableEvents();

        assertEquals(2, availableEvents.size());
        verify(eventRepository, times(1)).findBySeatsBookedLessThan(100);
    }

    @Test
    void getEventsBySpeaker() {
        when(eventRepository.findBySpeaker("John Doe")).thenReturn(Arrays.asList(event1));

        List<Event> events = eventService.getEventsBySpeaker("John Doe");

        assertEquals(1, events.size());
        verify(eventRepository, times(1)).findBySpeaker("John Doe");
    }

    @Test
    void createEvent() {
        when(eventRepository.save(event1)).thenReturn(event1);

        Event createdEvent = eventService.createEvent(event1);

        assertNotNull(createdEvent);
        assertEquals("Tech Conference 2024", createdEvent.getName());
        verify(eventRepository, times(1)).save(event1);
    }

    @Test
    void updateEvent() {
        when(eventRepository.existsById(1L)).thenReturn(true);
        when(eventRepository.save(event1)).thenReturn(event1);

        Event updatedEvent = eventService.updateEvent(event1);

        assertNotNull(updatedEvent);
        assertEquals("Tech Conference 2024", updatedEvent.getName());
        verify(eventRepository, times(1)).existsById(1L);
        verify(eventRepository, times(1)).save(event1);
    }

    @Test
    void bookSeats() {
        when(eventRepository.findById(1L)).thenReturn(Optional.of(event1));

        String result = eventService.bookSeats(1L, 10);

        assertEquals("Seats booked successfully", result);
        verify(eventRepository, times(1)).findById(1L);
    }

    @Test
    void deleteEvent() {
        when(eventRepository.findById(1L)).thenReturn(Optional.of(event1));

        eventService.deleteEvent(1L);

        verify(eventRepository, times(1)).delete(event1);
    }
}
2. Unit Tests for Controller Layer (EventController)
We will mock the EventService and test the controller endpoints.

EventControllerTest.java:
java
Copy code
package com.example.eventmanagement.controller;

import com.example.eventmanagement.entity.Event;
import com.example.eventmanagement.service.EventService;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;

import java.util.Arrays;

import static org.mockito.Mockito.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;

class EventControllerTest {

    private MockMvc mockMvc;

    @Mock
    private EventService eventService;

    @InjectMocks
    private EventController eventController;

    private Event event1;
    private Event event2;

    @BeforeEach
    void setUp() {
        MockitoAnnotations.openMocks(this);
        mockMvc = MockMvcBuilders.standaloneSetup(eventController).build();

        event1 = new Event();
        event1.setEventId(1L);
        event1.setName("Tech Conference 2024");
        event1.setSpeaker("John Doe");
        event1.setSeatsBooked(50);

        event2 = new Event();
        event2.setEventId(2L);
        event2.setName("AI Summit 2024");
        event2.setSpeaker("Jane Smith");
        event2.setSeatsBooked(10);
    }

    @Test
    void getAllEvents() throws Exception {
        when(eventService.getAllEvents()).thenReturn(Arrays.asList(event1, event2));

        mockMvc.perform(get("/event/events"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.length()").value(2))
                .andExpect(jsonPath("$[0].name").value("Tech Conference 2024"))
                .andExpect(jsonPath("$[1].name").value("AI Summit 2024"));

        verify(eventService, times(1)).getAllEvents();
    }

    @Test
    void getEventById() throws Exception {
        when(eventService.getEventById(1L)).thenReturn(event1);

        mockMvc.perform(get("/event/1"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name").value("Tech Conference 2024"));

        verify(eventService, times(1)).getEventById(1L);
    }

    @Test
    void createEvent() throws Exception {
        when(eventService.createEvent(any(Event.class))).thenReturn(event1);

        mockMvc.perform(post("/event")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content("{\"name\":\"Tech Conference 2024\", \"speaker\":\"John Doe\", \"seatsBooked\": 0}"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name").value("Tech Conference 2024"));

        verify(eventService, times(1)).createEvent(any(Event.class));
    }

    @Test
    void updateEvent() throws Exception {
        when(eventService.updateEvent(any(Event.class))).thenReturn(event1);

        mockMvc.perform(put("/event")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content("{\"eventId\":1,\"name\":\"Tech Conference 2024\", \"speaker\":\"John Doe\", \"seatsBooked\": 50}"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name").value("Tech Conference 2024"));

        verify(eventService, times(1)).updateEvent(any(Event.class));
    }

    @Test
    void bookSeats() throws Exception {
        when(eventService.bookSeats(1L, 10)).thenReturn("Seats booked successfully");

        mockMvc.perform(put("/event/book/1/10"))
                .andExpect(status().isOk())
                .andExpect(content().string("Seats booked successfully"));

        verify(eventService, times(1)).bookSeats(1L, 10);
    }

    @Test
    void deleteEvent() throws Exception {
        doNothing().when(eventService).deleteEvent(1L);

        mockMvc.perform(delete("/event/1"))
                .andExpect(status().isOk())
                .andExpect(content().string("Event deleted successfully"));

        verify(eventService, times(1)).deleteEvent(1L);
    }
}
Key Concepts:
Mocking Service Layer: In EventServiceTest, the EventRepository is mocked, and we test the business logic within EventService. Similarly, in EventControllerTest, the EventService is mocked to test the controller's behavior.

Assertions: We use assertEquals, assertNotNull, etc., in the service layer test. In the controller test, we use jsonPath and content() to assert the response structure and values.

MockMvc: It is used for testing the controller by simulating HTTP requests and verifying responses.

Running Tests:
You can run these tests using Maven or an IDE like IntelliJ or Eclipse:

bash
Copy code
mvn test
These unit tests ensure the correctness of your service and controller layers without hitting the actual database.








