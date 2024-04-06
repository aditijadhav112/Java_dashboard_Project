# Java_dashboard_Project

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*bhushuyyi;

import java.util.ArrayList;j8uuiuj
import java.util.HashMap;
import java.util.List;
import java.util.Map;gyghg

@SpringBootApplication
public class SchedulerApplication {Niyati

    public static void main(String[] args) {
        SpringApplication.run(SchedulerApplication.class, args);
    }

}

@RestController
@RequestMapping("/appointments")
class AppointmentController {

    // In-memory data store for appointments
    private final Map<Integer, List<Appointment>> appointmentsMap = new HashMap<>();

    // Initialize with three service operators
    {
        for (int i = 0; i < 3; i++) {
            appointmentsMap.put(i, new ArrayList<>());
        }
    }

    // Book an appointment
    @PostMapping("/book")
    public ResponseEntity<String> bookAppointment(@RequestBody Appointment appointment) {
        int operatorId = appointment.getOperatorId();
        List<Appointment> appointments = appointmentsMap.get(operatorId);

        // Check if the slot is available
        for (Appointment existingAppointment : appointments) {
            if (existingAppointment.overlaps(appointment)) {
                return ResponseEntity.status(HttpStatus.BAD_REQUEST).body("Appointment slot is not available");
            }
        }

        appointments.add(appointment);
        return ResponseEntity.ok("Appointment booked successfully");
    }

    // Reschedule an appointment
    @PutMapping("/reschedule/{appointmentId}")
    public ResponseEntity<String> rescheduleAppointment(@PathVariable int appointmentId, @RequestBody Appointment newAppointment) {
        int operatorId = newAppointment.getOperatorId();
        List<Appointment> appointments = appointmentsMap.get(operatorId);

        for (Appointment existingAppointment : appointments) {
            if (existingAppointment.getId() == appointmentId) {
                for (Appointment app : appointments) {
                    if (app.getId() != appointmentId && app.overlaps(newAppointment)) {
                        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body("New appointment slot is not available");
                    }
                }
                existingAppointment.setStartTime(newAppointment.getStartTime());
                existingAppointment.setEndTime(newAppointment.getEndTime());
                return ResponseEntity.ok("Appointment rescheduled successfully");
            }
        }

        return ResponseEntity.notFound().build();
    }

    // Cancel an appointment
    @DeleteMapping("/cancel/{appointmentId}")
    public ResponseEntity<String> cancelAppointment(@PathVariable int appointmentId) {
        for (List<Appointment> appointments : appointmentsMap.values()) {
            for (Appointment appointment : appointments) {
                if (appointment.getId() == appointmentId) {
                    appointments.remove(appointment);
                    return ResponseEntity.ok("Appointment canceled successfully");
                }
            }
        }

        return ResponseEntity.notFound().build();
    }

    // Get all booked appointments of an operator
    @GetMapping("/operator/{operatorId}")
    public ResponseEntity<List<Appointment>> getOperatorAppointments(@PathVariable int operatorId) {
        List<Appointment> appointments = appointmentsMap.get(operatorId);
        if (appointments == null) {
            return ResponseEntity.notFound().build();
        }
        return ResponseEntity.ok(appointments);
    }

    // Get open slots of appointments for an operator
    @GetMapping("/operator/{operatorId}/open-slots")
    public ResponseEntity<List<TimeSlot>> getOperatorOpenSlots(@PathVariable int operatorId) {
        List<Appointment> appointments = appointmentsMap.get(operatorId);
        if (appointments == null) {
            return ResponseEntity.notFound().build();
        }

        List<TimeSlot> openSlots = new ArrayList<>();
        int startTime = 0;
        for (Appointment appointment : appointments) {
            if (appointment.getStartTime() > startTime) {
                openSlots.add(new TimeSlot(startTime, appointment.getStartTime()));
            }
            startTime = appointment.getEndTime();
        }
        if (startTime < 24) {
            openSlots.add(new TimeSlot(startTime, 24));
        }

        return ResponseEntity.ok(openSlots);
    }

}

class Appointment {
    private static int nextId = 1;

    private int id;
    private int operatorId;
    private int startTime;
    private int endTime;

    public Appointment(int operatorId, int startTime, int endTime) {
        this.id = nextId++;
        this.operatorId = operatorId;
        this.startTime = startTime;
        this.endTime = endTime;
    }

    public int getId() {
        return id;
    }

    public int getOperatorId() {
        return operatorId;
    }

    public int getStartTime() {
        return startTime;
    }

    public void setStartTime(int startTime) {
        this.startTime = startTime;
    }

    public int getEndTime() {
        return endTime;
    }

    public void setEndTime(int endTime) {
        this.endTime = endTime;
    }

    // Check if this appointment overlaps with another appointment
    public boolean overlaps(Appointment other) {
        return !(this.endTime <= other.startTime || other.endTime <= this.startTime);
    }
}

class TimeSlot {
    private int start;
    private int end;

    public TimeSlot(int start, int end) {
        this.start = start;
        this.end = end;
    }

    public int getStart() {
        return start;
    }

    public int getEnd() {
        return end;
    }

    @Override
    public String toString() {
        return start + "-" + end;
    }
}


