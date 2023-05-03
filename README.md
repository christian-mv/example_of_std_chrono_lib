# example_of_std_chrono_lib
This is just a quick example on how to use the std::chrono library in C++20. 


```cpp

#include <iostream>
#include <chrono>
#include <format>

/// shows how to print timestamp in different time zones.
void convertingToDiferentTimezone(long long msSinceUtcEpoch) {
    using namespace std::chrono;
//    using namespace std::literals;

    std::cout << "------ Demo: convertingToDiferentTimezone ----- " << std::endl;

    auto timePoint = system_clock::time_point{ milliseconds{ msSinceUtcEpoch } };
    auto as_utc = zoned_time{ locate_zone("UTC"), timePoint };
    auto as_perth = zoned_time{ locate_zone("Australia/Perth"), timePoint };


    std::cout << "timePoint: "<< timePoint << " | " << std::format("{:%X %Z}", timePoint)  << std::endl;
    std::cout << "as_utc: "<< as_utc << " | " << std::format("{:%X %Z}", as_utc) << " | " << as_utc.get_time_zone()->name() << std::endl;
    std::cout << "as_perth: "<< as_perth << " | " << std::format("{:%X %Z}", as_perth) << " | " << as_perth.get_time_zone()->name() << std::endl;
    std::cout << std::endl;
}

void extractingDateAndTimeInfo(long long msSinceUtcEpoch){
    using namespace std::chrono;
//    using namespace std::literals;

    std::cout << "------ Demo: extractingDateAndTimeInfo  ----- " << std::endl;

    auto tp = zoned_time{current_zone(), system_clock::now()}.get_local_time();
    auto dp = floor<days>(tp);
    year_month_day ymd{dp};
    hh_mm_ss time{floor<milliseconds>(tp-dp)};

    std::cout << "Year: "<< int(ymd.year()) << std::endl;
    std::cout << "Month: "<< unsigned{ymd.month()} << std::endl;
    std::cout << "Day: "<< unsigned{ymd.day()} << std::endl;
    std::cout << "Hour: "<< time.hours().count() << std::endl;
    std::cout << "Minute: "<< time.minutes().count() << std::endl;
    std::cout << "Seconds: "<< time.seconds().count() << std::endl;
    std::cout << "Milliseconds: "<< time.subseconds().count() << std::endl;

    // alternatively, you can get the number of milliseconds since midnight as follows
    auto int_ms = duration_cast<milliseconds>(tp-dp); // number of milliseconds since midnight
    std::cout << "Total milliseconds since midnight: "<< int_ms.count() <<"\n"<< std::endl;

}


void createTimePointsWithDateTimeParts(int w_year, int w_month, int w_day,
                                       int w_hours, int w_minutes, int w_seconds, int w_milliseconds){

    //https://stackoverflow.com/questions/31284994/most-elegant-way-to-combine-chronotime-point-from-hours-minutes-seconds-etc

    using namespace std::chrono;

    std::cout << "------ Demo: createTimePointsWithDateTimeParts ----- " << std::endl;

    system_clock::time_point dateTime =
        sys_days(year(w_year)
                       /month(w_month)
                       /day(w_day))
        + hours(w_hours)
        + minutes(w_minutes)
        + seconds(w_seconds)
        + milliseconds(w_milliseconds);

    std::cout << dateTime << '\n'<<std::endl;


}

bool is_valid_iana_id(const std::string& iana_id) {
    try {
        auto zone = std::chrono::locate_zone(iana_id);
        (void)zone; // like Q_UNUSED
        return true;
    } catch (const std::exception&) {
        return false;
    }
}

void extractingDateAndTimeInfoToDifferentTimeZone(long long msSinceUtcEpoch, const std::string& ianaId) {
    using namespace std::chrono;
    std::cout << "------ Demo: createTimePointsWithDateTimeParts ----- " << std::endl;

    // check tha the timezone id is valid
    if(!is_valid_iana_id(ianaId)){
        std::cout<<"Invalid iana id: "<<ianaId<<std::endl;
        return;
    }


    auto zone_ptr = std::chrono::locate_zone(ianaId);
    if (!zone_ptr) {
        std::cout << ianaId << " is a valid IANA time zone ID.\n";
        return;
    }

    auto utcTimePoint = system_clock::time_point{ std::chrono::milliseconds{ msSinceUtcEpoch } };
    auto myZonedTime = zoned_time{ std::chrono::locate_zone(ianaId), utcTimePoint }.get_local_time();

    auto dp = floor<days>(myZonedTime);
    year_month_day ymd{dp};
    hh_mm_ss time{floor<milliseconds>(myZonedTime-dp)};

    std::cout << "Year: "<< int(ymd.year()) << std::endl;
    std::cout << "Month: "<< unsigned{ymd.month()} << std::endl;
    std::cout << "Day: "<< unsigned{ymd.day()} << std::endl;
    std::cout << "Hour: "<< time.hours().count() << std::endl;
    std::cout << "Minute: "<< time.minutes().count() << std::endl;
    std::cout << "Seconds: "<< time.seconds().count() << std::endl;
    std::cout << "Milliseconds: "<< time.subseconds().count() << std::endl;

    // alternatively, you can get the number of milliseconds since midnight as follows
    auto int_ms = duration_cast<milliseconds>(myZonedTime-dp); // number of milliseconds since midnight
    std::cout << "Total milliseconds since midnight: "<< int_ms.count() <<"\n"<< std::endl;

}


int main(){
    long long msSinceUtcEpoch = 1651406100010; // timestamp given in milliseconds since epoch in UTC

    convertingToDiferentTimezone(msSinceUtcEpoch); // demo 1

    extractingDateAndTimeInfo(msSinceUtcEpoch); // demo 2

    extractingDateAndTimeInfoToDifferentTimeZone(msSinceUtcEpoch, "Australia/Perth"); // demo 3

    createTimePointsWithDateTimeParts(2023, 10, 1, 2, 30, 0, 10); // demo 4
    return 0;
}


```
