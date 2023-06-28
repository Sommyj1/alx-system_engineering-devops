## Postmortem

It seems that there was an issue on an isolated Ubuntu 14.04 container running an Apache web server in Nigeria at approximately 06:05 West African Time (WAT). The issue occurred upon the release of ALX's System Engineering.

## What is a Postmortem?
A postmortem after an outage is a process of analyzing what happened during the outage, identifying the root cause, and developing an action plan to prevent it from happening again. It involves gathering data and feedback from team members and stakeholders, and creating a detailed report that can be used to improve processes and prevent similar issues in the future. The goal of a postmortem is to learn from the outage and prevent it from happening again.

## Use Cases
Postmortems after outages can be used in a variety of industries, including technology, healthcare, finance, and more. They can help identify weaknesses in systems, improve communication and collaboration among teams, and prevent future outages.

## Importance

Postmortems after outages are important because they help organizations learn from their mistakes, identify areas for improvement, and prevent future outages. They can help build a culture of continuous improvement, where teams are constantly striving to do better and achieve greater success. They also provide an opportunity for stakeholders to assess the impact of the outage on customers and other stakeholders, and to develop strategies to mitigate future risks. Ultimately, postmortems can help organizations improve their overall resilience and agility.

## Debugging Process

Bug debugger Brennan (BDB... as in my actual initials... made that up on the spot, pretty
good, huh?) encountered the issue upon opening the project and being, well, instructed to
address it, roughly 19:20 PST. He promptly proceeded to undergo solving the problem.


1. Checked running processes using `ps aux`. Two `apache2` processes - `root` and `www-data` -
were properly running.


2. Looked in the `sites-available` folder of the `/etc/apache2/` directory. Determined that
the web server was serving content located in `/var/www/html/`.


3. In one terminal, ran `strace` on the PID of the `root` Apache process. In another, curled
the server. Expected great things... only to be disappointed. `strace` gave no useful
information.


4. Repeated step 3, except on the PID of the `www-data` process. Kept expectations lower this
time... but was rewarded! `strace` revelead an `-1 ENOENT (No such file or directory)` error
occurring upon an attempt to access the file `/var/www/html/wp-includes/class-wp-locale.phpp`.


5. Looked through files in the `/var/www/html/` directory one-by-one, using Vim pattern
matching to try and locate the erroneous `.phpp` file extension. Located it in the
`wp-settings.php` file. (Line 137, `require_once( ABSPATH . WPINC . '/class-wp-locale.php' );`).


6. Removed the trailing `p` from the line.


7. Tested another `curl` on the server. 200 A-ok!


8. Wrote a Puppet manifest to automate fixing of the error.

## Summation


In short, a typo. Gotta love'em. In full, the WordPress app was encountering a critical
error in `wp-settings.php` when tyring to load the file `class-wp-locale.phpp`. The correct
file name, located in the `wp-content` directory of the application folder, was
`class-wp-locale.php`.


Patch involved a simple fix on the typo, removing the trailing `p`.

## Prevention

This outage was not a web server error, but an application error. To prevent such outages moving forward, please keep the following in mind.
•	Test! Test test test. Test the application before deploying. This error would have arisen and could have been addressed earlier had the app been tested.
•	Status monitoring. Enable some uptime-monitoring service such as UptimeRobot to alert instantly upon outage of the website.
Note that in response to this error, I wrote a Puppet manifest 0-strace_is_your_friend.pp to automate fixing of any such identitical errors should they occur in the future. The manifest replaces any phpp extensions in the file /var/www/html/wp-settings.php with php.
But of course, it will never occur again, because we're programmers, and we never make errors! 😉

## Link to Postmortem

To access the postmortem for this project, please click the following link: https://medium.com/@joseph2blessing2015/postmortem-alx-project-1a357763488f



