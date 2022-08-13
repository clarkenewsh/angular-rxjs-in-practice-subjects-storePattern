RxJS In Practice Course Notes:

    RxJS Intro:
        - Observables provide support for passing messages between parts of your application. They are used frequently in Angular and are a technique for event handling, asynchronous programming, and handling multiple values.The observer pattern is a software design pattern in which an object, called the subject, maintains a list of its dependents, called observers, and notifies them automatically of state changes. This pattern is similar (but not identical) to the publish/subscribe design pattern.Observables are declarative —that is, you define a function for publishing values, but it is not executed until a consumer subscribes to it. The subscribed consumer then receives notifications until the function completes, or until they unsubscribe. An observable can deliver multiple values of any type —literals, messages, or events, depending on the context. The API for receiving values is the same whether the values are delivered synchronously or asynchronously. Because setup and teardown logic are both handled by the observable, your application code only needs to worry about subscribing to consume values, and when done, unsubscribing. Whether the stream was keystrokes, an HTTP response, or an interval timer, the interface for listening to values and stopping listening is the same. Because of these advantages, observables are used extensively within Angular, and for application development as well.

        - RxJS is a library for reactive programming using Observables, to make it easier to compose asynchronous or callback-based code. This project is a rewrite of Reactive-Extensions/RxJS with better performance, better modularity, better debuggable call stacks, while staying mostly backwards compatible, with some breaking changes that reduce the API surface

        - RxJS is a library for composing asynchronous and event-based programs by using observable sequences. It provides one core type, the Observable, satellite types (Observer, Schedulers, Subjects) and operators inspired by Array methods (map, filter, reduce, every, etc) to allow handling asynchronous events as collections.

        - The essential concepts in RxJS which solve async event management are:

            - Observable: represents the idea of an invocable collection of future values or events.
            - Observer: is a collection of callbacks that knows how to listen to values delivered by the Observable.
            - Subscription: represents the execution of an Observable, is primarily useful for cancelling the execution.
            - Operators: are pure functions that enable a functional programming style of dealing with collections with operations like map, filter, concat, reduce, etc.
            - Subject: is equivalent to an EventEmitter, and the only way of multicasting a value or event to multiple Observers.
            -Schedulers: are centralized dispatchers to control concurrency, allowing us to coordinate when computation happens on e.g. setTimeout or requestAnimationFrame or others.
        
    
        - Additional Learning Resources:
            - RxJs Docs - https://rxjs.dev/guide/overview
            - Angular Docs - https://angular.io/guide/observables
            - RxJs Book - https://softchris.github.io/books/rxjs/why-rxjs/
            - Angular Uni Blog RxJs- https://blog.angular-university.io/tag/rxjs/
            - ng-conf Youtube - https://www.youtube.com/c/ngconfonline/search?query=rxjs
            - Angular Uni Youtube - https://www.youtube.com/channel/UC3cEGKhg3OERn-ihVsJcb7A/search?query= 
            - Decoded Frontend YouTube - https://www.youtube.com/c/DecodedFrontend/search?query=rxjs



        - Streams of values examples:

            - Stream of click values (multiple value stream that emit values over time, that never completes): 
                document.addEventListener('click', evt => {
                    console.log(evt)
                })

            - Intervals example
                - Stream of setInterval (multiple value stream that emit values over time, that never completes):
                    let counter = 0;
                    setInterval(() => {
                        console.log(counter);
                        counter++;
                    }, 1000);

                - Stream of setTimeout (stream that emits values and completes):
                    setTimeout(() => {
                        console.log('finished...');
                    }, 3000);

        - Streams can be combined instead of using native callback api, like above (if we combined the above example we would have callback hell.)          

        - RxJS can solve this solution for us to combine streams. 

        - RxJS Observable is a blueprint for a stream & will only become a stream if we subscribe to it. Multiple streams can be implemented as below;

            const interval$ = interval(1000);

            interval$.subscribe(val => console.log('stream 1 => ' + val));
            interval$.subscribe(val => console.log('stream 2 => ' + val));


        - RxJS Observable click stream example running at the same time as a interval$ stream;
            const interval$ = timer(3000, 1000);

            interval$.subscribe(val => console.log('stream 1 => ' + val));
            
            const click$ = fromEvent(document, 'click');
            click$.subscribe(evt console.log(evt));


        - Core RxJS Concepts - Errors, completion & subscriptions
            - Error callback to handle erros within the stream (most often we handling data from a server if an error occurs)
            - Completed can be used to tell us when the stream has completed and the stream can no longer emit values. In this case the stream ended when the error was emitted. 
            - err & complete are either one or the other as per the Observable contract
            
            const click$ = fromEvent(document, 'click');

            click$.subscribe(
                evt console.log(evt),
                err => console.log(err),
                () => console.log('completed')
            );

            - To unsubscribe from a stream, we can use the unsubscribe() func as implement the below (in this case we unsubscribe after 5 seconds using the setTimeout function);
                const interval$ = timer(3000, 1000);

                const sub = interval$.subscribe(val => console.log('stream 1 => ' + val));

                setTimeout(() => sub.unsubscribe(), 5000);
 
        - Custom Observable can be created using Observable.create(). An 'observer' is what we use internally to implement an observable and enables us to emit values in the observable.

        - Building our own HTTP Observable using fetch API (promise based).

            const http$ = Observable.create(observer => {
                fetch('/api/courses')
                    .then(response => {
                        return response.json();
                    })
                    .then(body => {
                        observer.next(body);
                        observer.complete();
                    })
                    .catch(err => {
                        observer.error(err);
                    })
                });

                https$.subscribe(
                    courses => console.log(courses),
                    // 'noop' rxjs used to implement empty callback instead of using () => {}
                    noop,
                    () => console.log('completed)
                );

        

        
    RxJS Operator & Reactive Design:

        - RxJs Map operator can be used to transform the json payload in to an array of courses.
        - Example custom observable, RxJs Map operator used with pipe operator, utilizing a custom http observable stored in utils file for reusability:

           // utils file (createHttpObservable can be reused)
           export function createHttpObservable(url: string) {
               return Observable.create(observer => {
                fetch('/api/courses')
                    .then(response => {
                        return response.json();
                    })
                    .then(body => {
                        observer.next(body);
                        observer.complete();
                    })
                    .catch(err => {
                        observer.error(err);
                    })
                });
           }

           // home.component.ts file
           ngOnInit() {
               const http$ = createHttpObservable('/api/courses');

               const courses$ = http$ 
               pipe(
                   map(res => Object.values(res['payload']))
               );

               courses$.subscribe(
                   courses => console.log(courses),
                   noop,
                   () => console.log('completed')
               );
           }
        
        - Building Components with RxJs - Imperative Design
            - Example imperative approach using pipe, map, filter to filter beginner & advanced courses (not best practice to have multiple callback filter funcs in our subscribe like below, this is an Rxjs anti-pattern. RxJs's purpose is to prevent just this callback hell)

            // home.component.ts file

            beginnerCourses: Course[];
            advancedCourses: Course[];

            ngOnInit() {
                const http$ = createHttpObservable('/api/courses');

                const courses$ = http$ 
                pipe(
                    map(res => Object.values(res['payload']))
                );

                courses$.subscribe(
                    courses => {
                        this.beginnerCourses = courses
                        .filter(course => course.category == 'BEGINNER');

                        this.advancedCourses = courses
                        .filter(course => course.category == 'ADVANCED');
                    },
                    noop,
                    () => console.log('completed')
                );
            }

            // home.component.html file
            <div class="courses-panel">

                <h3>All Courses</h3>

                <mat-tab-group>

                    <mat-tab label="Beginners">

                        <courses-card-list
                                [courses]="beginnerCourses">

                        </courses-card-list>

                    </mat-tab>

                    <mat-tab label="Advanced">

                        <courses-card-list
                                [courses]="advancedCourses"

                        ></courses-card-list>

                    </mat-tab>

                </mat-tab-group>

            </div>


        - Building Components with RxJs - Reactive Design
            - Example Reactive approach best practice with RxJS to avoid callback hell & not have nested subscribe() calls.) defining the Observable as so 'beginnerCourses$: Observable<Course[]>; then subscribing to it using the async pipe in the html template'.
            - Subscription is handled in the html view using the async pipe

            // utils file (createHttpObservable can be reused)
            export function createHttpObservable(url: string) {
               return Observable.create(observer => {
                fetch('/api/courses')
                    .then(response => {
                        return response.json();
                    })
                    .then(body => {
                        observer.next(body);
                        observer.complete();
                    })
                    .catch(err => {
                        observer.error(err);
                    })
                });
           }

            // home.component.ts file

            beginnerCourses$: Observable<Course[]>;
            advancedCourses$: Observable<Course[]>;

            ngOnInit() {
                const http$: Observable<Course[]> = createHttpObservable('/api/courses');

                const courses$ = http$ 
                pipe(
                    map(res => Object.values(res['payload'])) << map through & convert course payload in to array of courses
                );

                this.beginnerCourses$ = courses$
                    .pipe(
                        map(courses => courses
                            .filter(courses => course.category == 'BEGINNER')) << map through & filter courses based on beginner property
                    );

                this.advancedCourses$ = courses$
                    .pipe(
                        map(courses => courses
                            .filter(courses => course.category == 'ADVANCED'))
                    );
            }


            // home.component.html file
            <div class="courses-panel">

                <h3>All Courses</h3>

                <mat-tab-group>

                    <mat-tab label="Beginners">

                        <courses-card-list
                                [courses]="beginnerCourses$ | async">

                        </courses-card-list>

                    </mat-tab>

                    <mat-tab label="Advanced">

                        <courses-card-list
                                [courses]="advancedCourses$ | async"

                        ></courses-card-list>

                    </mat-tab>

                </mat-tab-group>

            </div>

            - Sharing HTTP Responses with shareReplay Operator
                - shareReplay() operator can be used when we are subscribing with two instances of the a http observable (as shown above in the home.component.html file).

                - Example:
                // home.component.ts file

                beginnerCourses$: Observable<Course[]>;
                advancedCourses$: Observable<Course[]>;

                ngOnInit() {
                    const http$: Observable<Course[]> = createHttpObservable('/api/courses');

                    const courses$ = http$ 
                    pipe(
                        tap(() => console.log('HTTP request executed')), << tap operator used to produce side effects in observable chain (ie print console)
                        map(res => Object.values(res['payload'])), << map through & convert course payload in to array of courses
                        shareReplay() << used to share the http observable request across multiple subscribers.. like below
                    );

                    this.beginnerCourses$ = courses$
                        .pipe(
                            map(courses => courses
                                .filter(courses => course.category == 'BEGINNER')) << map through & filter courses based on beginner property
                        );

                    this.advancedCourses$ = courses$
                        .pipe(
                            map(courses => courses
                                .filter(courses => course.category == 'ADVANCED'))
                        );
                }
            
            - Observable Concatenation
                - Observable Concatenation enables us to join multiple observables together. When the fist observable completes it moves on to the next (the observable must complete to move on to the other concatenated observables).
                - Browser Network log can be used to display the requests running in the network log to the backend

                - Example:
                ngOnInit() {
                    const source1$ = of(1, 2, 3);

                    const source2$ = of(4, 5, 6);

                    const source3$ = of(7, 8, 9);

                    const $result = concat(source1$, source2$, source3$);

                    result$.subscribe(console.log);
                }
            
            - RxJS concatMap & Filter Operator
                - concatMap can be used to to prevent multiple nested subscribes running at the same time.
                - Browser Network log can be used to display the requests running in the network log to the backend
                    - Use case below were we have 2 subscribes, 1 for subscribing to the form group valueChanges & 1 for saving the updated form values to the backend. concatMap can be used to wait for one save observable to complete before running another so we dont have overlapping request to the backend when a keyup event chnages the value of the form fields. Each key up tracks the change and then saves it to the backend without the subscrtions running at the same time.
                - Filter can be used on a formGroup to filter out form values that are invalid from the observable stream

                - Example of both concepts:
                
                // course-dialog.component.ts
                export class CourseDialogComponent implements OnInit, AfterViewInit {

                form: FormGroup;
                course:Course;

                @ViewChild('saveButton', { static: true }) saveButton: ElementRef;

                @ViewChild('searchInput', { static: true }) searchInput : ElementRef;

                constructor(
                    private fb: FormBuilder,
                    private dialogRef: MatDialogRef<CourseDialogComponent>,
                    @Inject(MAT_DIALOG_DATA) course:Course ) {

                    this.course = course;

                    this.form = fb.group({
                        description: [course.description, Validators.required],
                        category: [course.category, Validators.required],
                        releasedAt: [moment(), Validators.required],
                        longDescription: [course.longDescription,Validators.required]
                    });

                }

                ngOnInit() {

                    this.form.valueChanges
                    .pipe(
                        filter(() => this.form.valid),
                        concatMap(changes => this.saveCourse(changes))
                    )
                    .subscribe();
                }

                saveCourse(changes) {
                    return fromPromise(fetch(`/api/courses/${this.course.id}`, {
                        method: 'PUT',
                        body: JSON.stringify(changes),
                        headers: {
                            'content-type: 'application/json'
                        }
                    }));
                    
                }
                
                - merge & mergeMap Observable
                    - merge is used for performing asynchronous operations in parallel. Take the value from the first observable merge it with a second observable and provide a final value.
                    - Browser Network log can be used to display the requests running in the network log to the backend

                    - merge Example:
                    ngOnInit() {
                        const interval1$ = interval(1000);
                        const interval2$ = interval1$.pipe(map(val => 10 * val));
                        const result$ = merge(interval1$, interval2$);

                        result$.subscribe(console.log);
                    }

                    - mergeMap is ideal for performing HTTP Request in parallel. They emit the values as they are returned passes it to the next using the values from the previous observable. (for the autosave form fields example this is not best practice)

                    mergeMap Example:
                    // course-dialog.component.ts
                    export class CourseDialogComponent implements OnInit, AfterViewInit {

                    form: FormGroup;
                    course:Course;

                    @ViewChild('saveButton', { static: true }) saveButton: ElementRef;

                    @ViewChild('searchInput', { static: true }) searchInput : ElementRef;

                    constructor(
                        private fb: FormBuilder,
                        private dialogRef: MatDialogRef<CourseDialogComponent>,
                        @Inject(MAT_DIALOG_DATA) course:Course ) {

                        this.course = course;

                        this.form = fb.group({
                            description: [course.description, Validators.required],
                            category: [course.category, Validators.required],
                            releasedAt: [moment(), Validators.required],
                            longDescription: [course.longDescription,Validators.required]
                        });

                    }

                    ngOnInit() {

                        this.form.valueChanges
                        .pipe(
                            filter(() => this.form.valid),
                            mergeMap(changes => this.saveCourse(changes))
                        )
                        .subscribe();
                    }

                    saveCourse(changes) {
                        return fromPromise(fetch(`/api/courses/${this.course.id}`, {
                            method: 'PUT',
                            body: JSON.stringify(changes),
                            headers: {
                                'content-type: 'application/json'
                            }
                        }));
                        
                    }


                    - exhaustMap Operator
                        - exhaustMap can be used to ignore multiple parallel calls to the backend. For instance, if we have the form save button and the user clicks the save button multiple times, we want to ignore them multiple save calls.
                        - Browser Network log can be used to display the requests running in the network log to the backend

                        - exhaustMap Example:
                        export class CourseDialogComponent implements OnInit, AfterViewInit {

                        form: FormGroup;
                        course:Course;

                        @ViewChild('saveButton', { static: true }) saveButton: ElementRef;

                        @ViewChild('searchInput', { static: true }) searchInput : ElementRef;

                        constructor(
                            private fb: FormBuilder,
                            private dialogRef: MatDialogRef<CourseDialogComponent>,
                            @Inject(MAT_DIALOG_DATA) course:Course ) {

                            this.course = course;

                            this.form = fb.group({
                                description: [course.description, Validators.required],
                                category: [course.category, Validators.required],
                                releasedAt: [moment(), Validators.required],
                                longDescription: [course.longDescription,Validators.required]
                            });

                        }

                        ngOnInit() {

                            this.form.valueChanges
                            .pipe(
                                filter(() => this.form.valid),
                                concatMap(changes => this.saveCourse(changes))
                            )
                            .subscribe();
                        }

                        saveCourse(changes) {
                            return fromPromise(fetch(`/api/courses/${this.course.id}`, {
                                method: 'PUT',
                                body: JSON.stringify(changes),
                                headers: {
                                    'content-type: 'application/json'
                                }
                            }));
                            
                        }

                        ngAfterViewInit() {
                            fromEvent(this.saveButton.nativeElement, 'click')
                            .pipe(
                                exhaustMap(() => this.saveCourse(this.form.value))
                            )
                            .subscribe();
                        }
                    
                    - Unsubscription Observable
                        - Used to unsubscribe from the observable so to stop the stream from emitting values. This can be implemented with the ngOnDestroy() component lifecycle method.
                        - The fetch API comes with an 'AbortController' and is also a way to unsubscribe from a custom HTTP Observable.
                        - Using the Browser Dev Tools Network tab can be used to check the http request was cancelled.

                        - Example:

                            // about.component.ts 

                            - ngOnit() {
                                const http$ = createHttpObservable('/api/courses');

                                http$.subscribe(console.log);

                                setTimeout(() => {
                                    sub.unsubscribe() << the unsubscribe will trigger the return controller.abort() in the http observable
                                },0);
                            }

                           
                            // utils.ts file (createHttpObservable)

                            export function createHttpObservable(url: string) {
                            return Observable.create(observer => {

                                const controller = new AbortController();
                                const signal = controller.signal;    

                                fetch('/api/courses')
                                    .then(response => {
                                        return response.json();
                                    })
                                    .then(body => {
                                        observer.next(body);
                                        observer.complete();
                                    })
                                    .catch(err => {
                                        observer.error(err);
                                    });

                                    return () => controller.abort()
                                }); 
                            }
                        

                        - SwitchMap Operator
                            - switchMap Operator use case could be to implement a Search Typeahead feature. This can be done observing a stream of events using the keyup event. Switch map enables us to make a http observable request when a search term has been provided and sent to the backend, but then the users starts adding extra search terms, switch map then unsubscribes, switches to a new http observable request containing the new values and sends that value to the backend. 

                            - debounceTime operator can be used to reduce the number of search requests produced from every key up event, that will be sent to the backend (i.e dont send a request to the backend every key press). debounce time can be used to specify a an delay/interval, i.e 20ms between keypress. After the 20ms send the value in the input search field to the backend. A stable value when using deboucne time has elapsed and the value left in the search field is classed as the stable value. That value is the send as the search parameter to the backend. 
                                                        
                            - distinctUntilChange can be used to emit duplicate values (duplicate values are emitted when we press the alt key with a letter for a uppercase letter).

                            - Angular Uni Video Link for this in action 
                                - https://www.udemy.com/course/rxjs-course/learn/lecture/10810514#overview

                            - Example of typeAhead feature using switchMap, debouceTime, distinctUntilChange & concatMap with custom http observable:

                              // utils.ts file (createHttpObservable)

                            export function createHttpObservable(url: string) {
                            return Observable.create(observer => {

                                const controller = new AbortController();
                                const signal = controller.signal;    

                                fetch('/api/courses')
                                    .then(response => {
                                        return response.json();
                                    })
                                    .then(body => {
                                        observer.next(body);
                                        observer.complete();
                                    })
                                    .catch(err => {
                                        observer.error(err);
                                    });

                                    return () => controller.abort()
                                }); 
                            }
                            
                            // course.component.html
                            <div class="course">

                                <ng-container *ngIf="(course$ | async) as course">

                                    <h2>{{course?.description}}</h2>

                                    <img class="course-thumbnail" [src]="course?.iconUrl">

                                </ng-container>

                                <mat-form-field class="search-bar">

                                    <input matInput placeholder="Type your search" #searchInput autocomplete="off">

                                </mat-form-field>


                                <table class="lessons-table mat-elevation-z7" *ngIf="(lessons$ | async) as lessons">

                                    <thead>
                                        <th>#</th>
                                        <th>Description</th>
                                        <th>Duration</th>
                                    </thead>

                                    <tr *ngFor="let lesson of lessons">
                                        <td class="seqno-cell">{{lesson.seqNo}}</td>
                                        <td class="description-cell">{{lesson.description}}</td>
                                        <td class="duration-cell">{{lesson.duration}}</td>
                                    </tr>

                                </table>

                            </div>


                            // course.component.ts

                            export class CourseComponent implements OnInit, AfterViewInit {

                                courseId:string;

                                course$ : Observable<Course>;

                                lessons$: Observable<Lesson[]>;


                                @ViewChild('searchInput', { static: true }) input: ElementRef; << get reference to the search input field

                                constructor(private route: ActivatedRoute) {

                            }

                             ngOnInit() {

                                this.courseId = this.route.snapshot.params['id'];

                                this.course$ = createHttpObservable(`/api/courses/${this.courseId}`);

                                this.lessons$ = this.loadLessons(); 
                            }


                            ngAfterViewInit() {
                            searchLessons$ = fromEvent<any>(this.input.nativeElement, 'keyup')
                                .pipe(
                                    map(event => event.target.value),
                                    debounceTime(400),
                                    distinctUntilChanged(),
                                    switchMap(search => this.loadLessons(search))
                                );

                                const intialLessons$ = this.loadLessons();
                                
                                this.lessons$ = concat(initialLessons$, searchLessons$)

                            }

                            loadLessons(search = ''): Observable<Lesson[]> {
                                return createHttpObservable(
                                    `/api/lessons?courseId=${this.courseId}&pageSize=100&filter=${search}`)
                                    .pipe(
                                        map(res => res["payload"])
                                    );
                            }


    RxJs Error Handling:
        - rxjs error handling is important strategy which enables us to handle any http observable stream errors.
        - errors can be caught and provided with an alternative value to proceed with (something shown in the UI as an alternative) using the below implementation: 
            - catchError(err => of([
                id: 0,
                description: "RxJs In Practice Course",
                iconUrl: 'https://s3-us-west-1.amazonaws.com/angular-university/course-images/rxjs-in-practice-course.png',
                courseListIcon: 'https://angular-academy.s3.amazonaws.com/main-logo/main-page-logo-small-hat.png',
                longDescription: "Understand the RxJs Observable pattern, learn the RxJs Operators via practical examples",
                category: 'BEGINNER',
                lessonsCount: 10
            ]))

        - errors can be caught and rethrow the error code to the console using Catch & Rethrow strategy, using the below:
            - catchError(err => {
                console.log('Error occurred, err');

                return throwError(err);
                }
            ) 
        
        - finalize can be invoked to finalise an error using the below
            - finalize(() => {
                console.log('error finalised');
            })

        - errors can be retired.
        - catch & replace error handling strategy can be used.
        - catch & retry strategy can be used using the below (i.e retry after 2 seconds after initial error and try again):
             retryWhen(error => error.pipe(
                 delayWhen(() => timer(2000)) << wait for 2 seconds to try again
             )).
        
        - startWith operator can be used to intilaise a stream with an initial value - startWith('');

        - Throttling vs Debouncing operators are provide similar functionality. Debouncing is about waiting for a value to become stable, for example waiting 4 seconds to ensure a value in a search field has become stable before sending the request to the backend. Throttling on the other hand is used limit the number of values that can be emitted during a certain interval. Throttling can be implemented as so 
            - throttle(() => interval(500));


    RxJS Custom Operator:
        - Custom Rxjs debug operators can be used to store error logging message/statements to help us debug any issues we come across with rxjs observables.
        - Example of custom debug operator:
            
            // debug.ts file
            import {Observable} from 'rxjs';
            import {tap} from 'rxjs/operators';

            export enum RxJsLoggingLevel {
                TRACE,
                DEBUG,
                INFO,
                ERROR
            }

            let rxjsLoggingLevel = RxJsLoggingLevel.INFO;

            export function setRxJsLoggingLevel(level: RxJsLoggingLevel) {
                rxjsLoggingLevel = level;
            }


            export const debug = (level: number, message:string) =>
                (source: Observable<any>) => source
                    .pipe(
                        tap(val => {

                            if (level >= rxjsLoggingLevel) {
                                console.log(message + ': ', val);
                            }
                        })
                    );

            // course.component.ts file
            export class CourseComponent implements OnInit, AfterViewInit {

            courseId:string;

            course$ : Observable<Course>;

            lessons$: Observable<Lesson[]>;


            @ViewChild('searchInput', { static: true }) input: ElementRef;

            constructor(private route: ActivatedRoute) {


            }

            ngOnInit() {

                this.courseId = this.route.snapshot.params['id'];

                this.course$ = createHttpObservable(`/api/courses/${this.courseId}`)
                    .pipe(
                        debug( RxJsLoggingLevel.INFO, "course value "),
                    );

                setRxJsLoggingLevel(RxJsLoggingLevel.TRACE);

            }

            ngAfterViewInit() {

                this.lessons$ = fromEvent<any>(this.input.nativeElement, 'keyup')
                    .pipe(
                        map(event => event.target.value),
                        startWith(''),
                        debug( RxJsLoggingLevel.TRACE, "search "),
                        debounceTime(400),
                        distinctUntilChanged(),
                        switchMap(search => this.loadLessons(search)),
                        debug( RxJsLoggingLevel.DEBUG, "lessons value ")
                    );

            }

            loadLessons(search = ''): Observable<Lesson[]> {
                return createHttpObservable(
                    `/api/lessons?courseId=${this.courseId}&pageSize=100&filter=${search}`)
                    .pipe(
                        map(res => res["payload"])
                );
            }


        }

        - The RxJs forkJoin Operator allows us to launch several task in parallel, wait for the task to complete and then get back the results of each task and used the combined tasks together.




    RxJS Subjects & Custom Store Pattern:
        - •	A Subject is a stream of data - it can have multiple subscribers which receive the data that is emitted from the subject. So every time the code says subject.next(data), the subscribers will receive the passed data. Subject: is equivalent to an EventEmitter, and the only way of multi-casting a value or event to multiple Observers. Subject should be set to private to the part of the application that is emitting the data within the component. Public observables can then be used to use outside the component (like our example below).

        - A Subject is like an Observable, but can multicast to many Observers. Subjects are like EventEmitters: they maintain a registry of many listeners. Every Subject is an Observable. Given a Subject, you can subscribe to it, providing an Observer, which will start receiving values normally. From the perspective of the Observer, it cannot tell whether the Observable execution is coming from a plain unicast Observable or a Subject.

        - We don't expose the 'subjects' directly to store clients, instead, we expose an observable. This is to prevent the service clients from themselves emitting store values directly instead of calling action methods and therefore bypassing the store.Exposing the subject directly could lead to event soup applications, where events get chained together in a hard to reason way. Direct access to an internal implementation detail of the subject is like returning internal references to internal data structures of an object: exposing the internal means yielding the control of the subject and allowing for third parties to emit values. There might be valid use cases for this, but this is most likely almost never what is intended.

        - •	BehaviorSubjects are a little different to subjects as  we can utilize a default / start value that we can show initially if it takes some time before the first values starts to arrive. We can inspect the latest emitted value and of course listen to everything that has been emitted. One of the variants of Subjects is the BehaviorSubject, which has a notion of "the current value". It stores the latest value emitted to its consumers, and whenever a new Observer subscribes, it will immediately receive the "current value" from the BehaviorSubject.

        - The heart of an observable data service is the RxJs Subject. Subjects implement both the Observer and the Observable interfaces, meaning that we can use them to both emit values and register subscribers. The subject is nothing more than a traditional event bus, but much more powerful as it provides all the RxJs functional operators with it. But at its heart, we simply use it to subscribe just like a regular observable:

        - There is another property of the BehaviorSubject that is interesting: we can at any time retrieve the current value of the stream: let currentValue = behaviorSubject.getValue();

        - BehaviorSubjects are useful for representing "values over time". For instance, an event stream of birthdays is a Subject, but the stream of a person's age would be a BehaviorSubject. They should be set to private as as follows ‘private _currentOpenMenu$ = new BehaviorSubject<MenuType>(initial value here) and then call .next(value), then using the async pipe we can subscribe to and every subscriber will get that behavior subject value. We set it to private as the behavior subject is the one creating values, we then expose then values publicly with, this.currentOpenMenu$ = this._currentOpenMenu$.asObservable();

        - Subjects & BehaviorSubjects - Is a strategy we use them to emit values. For example, service with a subject is were we pass new values to the subject and multiple components can subscribe to get the emitted values from the subject. 

        - Example with BehaviorSubject
            o	yeah so every time we click to open a menu, the click event should call a method that takes the MenuType.SelectedMenu, (MenuType store in TS Enum, were each value represents a different component on the side menu), then in that method the subject.nex(selectedMenu) should be called
            o	onces the .next fires, every subscriber to the subject will immediately get that selectedMenu value
            o	think we have only one subscriber, which is used in the html file, by using the async pipe

        - BehaviorSubject Example:

            // Component file	
            private _currentOpenMenu$ = new BehaviorSubject<MenuType>(intialValue)
                public currentOpenMenu$: Observable<MenuType>
                constructor() {
                    this.currentOpenMenu$ = this._currentOpenMenu$.asObservable();
            }

            Toggle(MenuTypeValue) {
                This.currentOpenMenu$.next(MenuTypeValue);
            }

            // template file
            <div *ngIf=”(currentOpenMenu$ | async) as currentOpenMenu””>
                <button click=”Toggle(MenuTypeValue)” [ngClass]=”currentOpenMenu === MenuTypeValue >select</button>
            </div>

        - •	AsyncSubject is ideal for using with long running calculations that is emitting a lot of values over time. It then emits the last value emitted after calculating all off the emitted values and the subject is then complete. The AsyncSubject is a variant where only the last value of the Observable execution is sent to its observers, and only when the execution completes. 

        - •	The ReplaySubject is going to replay the complete observable, emitting all of the values emitted. It does not need to complete like the asyncSubject. A ReplaySubject is similar to a BehaviorSubject in that it can send old values to new subscribers, but it can also record a part of the Observable execution.

        - The Course Store Pattern implemented in the tutorial video is a great example and use case of subject and Rxjs in general - refer to the store.service.ts file to see the implementation of the store. The store enables us to hold state within our application which can then be tapped into by other parts of the application by subscribing to the observable data in the store. By having a store we don't have to have multiple calls to the backend to retrieve data on each page load when moving around the application views. In most cases the stateless approach is sufficient but by having a store that stores stateful data we can improve performance.   
        
        - Force completion of long running observables using the 'First' & 'Take' Operators.

        - withLatestFrom RxJs Operator is used to combine multiple observables together by taking the latest value emitted from each observable and providing the emitted value to the next operator in the chain or subscribe method as a 'tuple value' (A Tuple is an array with special properties and considerations: The number of elements of the array i s fixed ( aka immutable ).

    