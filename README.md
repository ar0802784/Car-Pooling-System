# RidePool

Find people travelling the same way, split the fare, and see the cheapest option across Ola, Uber, Rapido and inDrive.

Built with Java 17, Spring Boot 3, Thymeleaf, Spring Data JPA, H2 (or MySQL), and Leaflet + OpenStreetMap for maps.

## Run it

You need **JDK 17 or newer** and **Maven 3.8+**. An internet connection is needed for the map tiles and place search.

```bash
cd ridepool
mvn spring-boot:run
```

Open http://localhost:8080 and create an account. The database is created automatically in `./data/`.

In IntelliJ, Eclipse or VS Code: open the folder as a Maven project and run `RidePoolApplication`.

## Demo in two minutes

1. Register and log in. You land on the ride request page.
2. Type a pickup and a drop, pick each from the suggestion list (or tap the map), choose a time, then press **Find riders**.
3. The searching page shows your route, the estimated distance, time and cost, and a map that checks for riders every 4 seconds.
4. Press **Add demo riders nearby**. Three pretend riders are created near your route and the page moves on by itself.
5. The results page shows the best deal, each co-rider's phone, email and rating, and the fare comparison.
6. Press **Message** on a rider's card to chat. Demo riders reply automatically, so you can show the chat on one laptop.
7. Pick a service in the fare table (the best deal is pre-selected) and press **Book now** to see the booking summary and open that service's app.
8. To see the "no match" path, press **Skip waiting and see the best deals** instead. You still get the cheapest option.

For a two-person demo with real accounts, register two users in two browsers (or one normal and one private window), and enter nearly the same pickup, drop and time.

## How the pieces work

| Step | Where |
|------|-------|
| Login, register, logout | `web/AuthController`, `web/AuthInterceptor` (session based, BCrypt passwords) |
| Ride request, searching, results | `web/RideController` and the templates in `resources/templates` |
| Matching | `service/MatchService` |
| Distance and time | `service/GeoService` (Haversine formula) |
| Fare estimates and best deal | `service/FareService` |
| Demo riders | `service/DemoService` |
| In-app chat and inbox | `service/MessageService`, `web/ChatController`, `chat.html`, `messages.html` |
| Choose a service and Book now | `web/BookingController`, `results.html`, `booking.html` |

**Matching rule.** Two ride requests match when the pickups are within 2 km, the drops are within 2 km, and the departure times are within 20 minutes. Change these in `application.properties`.

**Fare estimates.** Ola, Uber, Rapido and inDrive do not offer public fare APIs, so the app estimates each fare as `max(minFare, base + perKm * km + perMin * minutes)`, plus a peak-hour increase (8-10 am and 5-8 pm). The rates live in the `RATES` list in `FareService.java`. Cabs split the fare among up to 4 riders; the Rapido bike taxi carries one person, so it is never split. The best deal is whichever option costs you the least per person. Describe these as estimates in your report and demo.

**Chat.** You can message someone while your ride requests match, and the chat stays open once you have exchanged messages. The unread count shows next to Messages in the top bar. Pages check for new messages every 3 seconds.

**Book now.** Ola, Uber, Rapido and inDrive give other apps no way to place a booking, so Book now saves your choice (service, vehicle, total fare, your share), shows a summary with the pickup and drop to copy, lets you message co-riders with a ready-made note, and opens the service's website. The final booking and payment happen in that app. The fare is recalculated on the server when you press Book now.

**Distance.** Road distance is the straight-line distance times 1.3. The map draws the real road route using the free OSRM demo server, but costs use the estimate so the app still works if that server is slow.

## Switching to MySQL

Add this to `pom.xml`:

```xml
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <scope>runtime</scope>
</dependency>
```

Then use the MySQL lines in `application.properties` (they are commented in the file).

## Ideas to extend it

- Real-time updates with WebSockets instead of polling
- Send a push or email when a new chat message arrives
- Mark a ride as completed and only allow ratings after that
- Route-overlap matching, so riders can join along the way
- Admin page that lets you edit fare rates
- Email or SMS notification when a match is found
