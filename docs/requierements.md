# System Zarządzania Nauką (LMS) — Wymagania

## 1. Cel i kontekst
Celem jest stworzenie webowego LMS w .NET 8 + SQL Server z EF Core. System umożliwia prowadzenie kursów, zapisy studentów oraz obsługę zadań i ocen. Projekt spełnia wymagania: role, ≥5 funkcji, CRUD, autoryzacja, testy, dokumentacja, obiekty DB (procedury, funkcja, trigger). [BDwAI projekt] 

## 2. Role użytkowników
- **Admin** — zarządza użytkownikami, nadzoruje kursy, generuje raporty globalne.
- **Teacher** — tworzy kursy/lekcje/zadania, akceptuje zapisy, ocenia, generuje raporty kursu.
- **Student** — przegląda kursy, zapisuje się, realizuje lekcje, składa rozwiązania zadań.

## 3. Główne funkcje (min. 5)
1) Rejestracja i logowanie (JWT/Identity).  
2) Zarządzanie kursami (CRUD) — Teacher/Admin.  
3) Zapisy na kurs i akceptacja (Student → Pending; Teacher/Admin → Approve).  
4) Lekcje i zadania (CRUD), terminy oddania.  
5) Składanie rozwiązań (Submissions) + wystawianie punktów.  
6) Raport kursu (liczba zapisów, średnia punktów, terminowość).  
7) Logowanie działań (ActivityLogs) i błędów.

## 4. Procesy biznesowe (end-to-end)
**P1: Zapis na kurs**
- Student → `POST /enrollments` (Status=Pending)  
- Teacher/Admin → `POST /enrollments/{id}/approve` (Status=Approved; wpis w ActivityLogs)  
- Student uzyskuje dostęp do lekcji/zadań kursu.

**P2: Zadanie i ocena**
- Teacher tworzy Assignment (title, dueDate, maxPoints).  
- Student → `POST /assignments/{id}/submit` (plik/URL, timestamp).  
- Teacher wystawia punkty → Submission.Points.  
- Raport kursu uwzględnia terminowość (SubmittedAt ≤ DueDate) i średnią punktów.

## 5. Wymagania niefunkcjonalne
- Bezpieczeństwo: JWT, role-based auth; walidacja danych.  
- Jakość API: spójne kody błędów (ProblemDetails), paginacja list.  
- Testy: min. 5 sensownych testów (xUnit) — logika procesów, autoryzacja, walidacje.  
- Dokumentacja: Swagger UI + /docs (opis, ERD, obiekty DB, instrukcja).  
- Uruchamialność: `dotnet run`, `dotnet ef database update`.

## 6. API — szkic endpointów
- **Auth**: `POST /auth/register`, `POST /auth/login`  
- **Courses**: `GET /courses`, `GET /courses/{id}`, `POST /courses`*, `PUT /courses/{id}`*, `DELETE /courses/{id}`* (*Teacher/Admin)  
- **Lessons**: `GET /courses/{id}/lessons`, `POST /lessons`*, `PUT /lessons/{id}`*, `DELETE /lessons/{id}`*  
- **Assignments**: `GET /lessons/{id}/assignments`, `POST /assignments`*, `PUT /assignments/{id}`*, `DELETE /assignments/{id}`*  
- **Submissions**: `POST /assignments/{id}/submit` (Student), `PUT /submissions/{id}/grade` (Teacher/Admin)  
- **Enrollments**: `POST /enrollments` (Student), `POST /enrollments/{id}/approve` (Teacher/Admin)  
- **Reports**: `GET /courses/{id}/report?year=&month=` (Teacher/Admin)

## 7. Model danych (szkic)
- **Users**(Id, Email, PasswordHash, Role, CreatedAt)  
- **Courses**(Id, Title, Description, TeacherId→Users, IsActive)  
- **Enrollments**(Id, CourseId→Courses, StudentId→Users, Status[Pending|Approved], EnrolledAt)  
- **Lessons**(Id, CourseId→Courses, Title, Content, PublishAt)  
- **Assignments**(Id, LessonId→Lessons, Title, DueDate, MaxPoints)  
- **Submissions**(Id, AssignmentId→Assignments, StudentId→Users, SubmittedAt, ContentUrl, Points NULL)  
- **ActivityLogs**(Id, UserId→Users, Action, Entity, EntityId, CreatedAt)

## 8. Obiekty bazy danych (do implementacji)
- **Procedury (≥2):**  
  - `sp_ApproveEnrollment(@EnrollmentId, @ReviewerId)` — ustawia Approved + log.  
  - `sp_CourseMonthlyReport(@CourseId, @Year, @Month)` — zwraca: liczba zapisów Approved, średnia punktów, % terminowych submissions.
- **Funkcja (≥1):**  
  - `ufn_IsEnrollmentAllowed(@CourseId, @StudentId)` → `bit` (kurs aktywny, brak duplikatu zapisu).  
- **Trigger (≥1):**  
  - `trg_Submissions_AfterInsert` — po INSERT do Submissions dodaje wpis do ActivityLogs.

## 9. Reguły i walidacje
- Enrollment wymaga `ufn_IsEnrollmentAllowed=1`.  
- DueDate > teraz; Submission po terminie dozwolone tylko z flagą „Late” (lub blokowane — decyzja projektowa).  
- Teacher może modyfikować tylko własne kursy; Student tylko własne submissions.

## 10. Kryteria akceptacji (przykłady)
- Student zapisany (Approved) widzi lekcje i może składać submissions.  
- Raport kursu zwraca poprawne agregaty dla wybranego miesiąca.  
- Zdarzenia kluczowe zapisują się w ActivityLogs (ApproveEnrollment, SubmitAssignment, GradeSubmission).  
- Swagger UI opisuje wszystkie endpointy; niechronione są tylko Auth.

## 11. Mapowanie na wymagania z karty przedmiotu
- Role + ≥5 funkcji + pełny proces + raporty + logowanie działań ✔  
- Backend (.NET + EF), CRUD + logika ✔  
- DB: ≥4 tabele, procedury, funkcja, trigger ✔  
- Frontend: min. 3 widoki (Auth, Kursy/Zapisy, Zadania/Submissions) ✔  
- Testy (≥5), Dokumentacja (/docs + Swagger) ✔
