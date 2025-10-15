# Welcome to my blog

I'm glad you are here. I plan to talk about ...
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Interactive Google Dorks Explorer</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Chosen Palette: Warm Neutrals with Muted Blue Accent -->
    <!-- Application Structure Plan: A dashboard-style layout was chosen to present the Google Dorks as a practical toolkit. The structure includes a prominent search bar and category filters for immediate interaction, a card-based grid for clear presentation of each dork, and a summary chart to provide an analytical overview. This design prioritizes user tasks—finding and using dorks—over a simple linear list, making the tool more efficient and user-friendly for security researchers and enthusiasts. The flow is designed for exploration: users can get a high-level view from the chart, then drill down using category filters or specific keywords. -->
    <!-- Visualization & Content Choices: 
        - Dorks List -> Goal: Organize & Use -> Presentation: Interactive Card Grid -> Interaction: Real-time search and category filtering -> Justification: Transforms a static list into a dynamic, searchable database, vastly improving usability.
        - Dork Categories -> Goal: Analyze & Navigate -> Viz: Bar Chart (Chart.js) -> Interaction: Hover tooltips -> Justification: Provides a quick visual summary of the dork types available, highlighting common vulnerability vectors and aiding navigation.
        - Individual Dork -> Goal: Use -> Presentation: Text with 'Copy' & 'Test' buttons -> Interaction: Click -> Justification: Facilitates direct use of the dorks, minimizing manual effort and potential for error. All visualizations are rendered on Canvas via Chart.js.
    -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <style>
        body {
            background-color: #f8f9fa;
            font-family: 'Inter', sans-serif;
        }
        .chart-container {
            position: relative;
            width: 100%;
            max-width: 800px;
            margin-left: auto;
            margin-right: auto;
            height: 400px;
            max-height: 50vh;
        }
        .card-enter {
            opacity: 0;
            transform: translateY(20px);
            transition: opacity 0.3s ease-out, transform 0.3s ease-out;
        }
        .card-enter-active {
            opacity: 1;
            transform: translateY(0);
        }
    </style>
</head>
<body class="text-gray-800">

    <div class="container mx-auto p-4 md:p-8">
        <header class="text-center mb-8">
            <h1 class="text-4xl md:text-5xl font-bold text-gray-900 mb-2">Google Dorks Explorer</h1>
            <p class="text-lg text-gray-600 max-w-3xl mx-auto">An interactive toolkit to discover, filter, and utilize powerful Google search queries for security research and reconnaissance.</p>
        </header>

        <main>
            <section id="dashboard" class="mb-8">
                <h2 class="text-2xl font-bold mb-4 text-center">Dork Categories Overview</h2>
                <p class="text-md text-gray-600 text-center mb-6 max-w-3xl mx-auto">This chart provides a high-level breakdown of the dorks in our collection by their primary purpose, such as finding exposed documents, configuration files, or login portals. Use it to understand the landscape of available queries before diving in.</p>
                <div class="bg-white p-6 rounded-xl shadow-lg">
                    <div class="chart-container">
                        <canvas id="dorksChart"></canvas>
                    </div>
                </div>
            </section>

            <section id="explorer">
                <div class="bg-white p-6 rounded-xl shadow-lg mb-8 sticky top-0 z-10">
                    <div class="grid grid-cols-1 md:grid-cols-3 gap-4 items-center">
                        <div class="md:col-span-2">
                            <label for="search-input" class="sr-only">Search Dorks</label>
                            <div class="relative">
                                <span class="absolute inset-y-0 left-0 flex items-center pl-3">
                                    <i class="fas fa-search text-gray-400"></i>
                                </span>
                                <input type="text" id="search-input" placeholder="Search by keyword (e.g., password, admin, .gov)..." class="w-full p-3 pl-10 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition">
                            </div>
                        </div>
                        <div class="relative">
                            <label for="category-filter" class="sr-only">Filter by Category</label>
                            <select id="category-filter" class="w-full p-3 border border-gray-300 rounded-lg appearance-none bg-white focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition">
                                <option value="all">All Categories</option>
                            </select>
                             <div class="pointer-events-none absolute inset-y-0 right-0 flex items-center px-2 text-gray-700">
                                <i class="fas fa-chevron-down"></i>
                            </div>
                        </div>
                    </div>
                </div>
                
                <div class="text-center mb-4">
                     <p id="results-count" class="text-lg text-gray-600"></p>
                </div>

                <div id="dorks-grid" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                </div>
                 <div id="no-results" class="hidden text-center py-16">
                    <i class="fas fa-eye-slash text-5xl text-gray-400 mb-4"></i>
                    <h3 class="text-2xl font-semibold text-gray-700">No Dorks Found</h3>
                    <p class="text-gray-500 mt-2">Try adjusting your search or filter criteria.</p>
                </div>
            </section>
        </main>
        
        <footer class="text-center mt-12 py-6 border-t border-gray-200">
            <p class="text-gray-500">&copy; 2025 Dorks Explorer. For educational and research purposes only.</p>
        </footer>
        
        <div id="toast" class="fixed bottom-5 right-5 bg-green-500 text-white py-2 px-4 rounded-lg shadow-xl opacity-0 transform translate-y-2 transition-all duration-300">
            <i class="fas fa-check-circle mr-2"></i>Copied to clipboard!
        </div>
    </div>

    <script>
        const rawDorksData = `
site:gov.* intitle:index of *.csv,Files Containing Juicy Info
Fwd: intitle:Index of / intext:resource/,Files Containing Juicy Info
Google to wordpress,Files Containing Juicy Info
Fwd: intitle:atvise - next generation,Files Containing Juicy Info
site:papaly.com + keyword,Files Containing Juicy Info
site:pastebin.com intitle:cpanel,Files Containing Juicy Info
intitle:index of settings.py,Pages Containing Login Portals
site:postman.com + keyword,This dork returns public postman API collections
inurl:admin filetype:xlsx site:gov.*,Files Containing Juicy Info
db_password filetype:env,Files Containing Juicy Info
inurl: /wp-content/uploads/ inurl:robots.txt Disallow: filetype:txt,Files Containing Juicy Info
inurl:admin filetype:xls,Files Containing Juicy Info
site:gov.* intitle:index of *.apk,Files Containing Juicy Info
inurl:admin filetype:txt,Files Containing Juicy Info
inurl:admin filetype:xls site:gov.in,Files Containing Juicy Info
intitle:index of /products,Files Containing Juicy Info
intitle:index of /mysql,Files Containing Juicy Info
site:*.ng intitle:index of,Files Containing Juicy Info
site:*.edu.in intitle:index of,Files Containing Juicy Info
inurl:*gov intitle:index of docker-compose,Files Containing Juicy Info
inurl:pastebin SHODAN_API_KEY,Files Containing Juicy Info
inurl:*gov intitle:index of/documents,Files Containing Juicy Info
intitle:index of php,Files Containing Juicy Info
intitle:index of site:gov.in,Files Containing Juicy Info
site:*.github.io intext:cheatsheet+offensive+pentesting,Files Containing Juicy Info
intitle:index of admin.js,Files Containing Juicy Info
inurl:gov.in & inurl:admin,Files Containing Juicy Info
intitle:index of wp-inc,Files Containing Juicy Info
allintext:account number,Files Containing Juicy Info
site:.edu intext:index of payroll,Files Containing Juicy Info
intitle:index of *.yaml,Files Containing Juicy Info
site:*.se intitle:index of,Files Containing Juicy Info
site:*.id intitle:index of screenshot*.jpg,Files Containing Juicy Info
intitle:index of *.vcf,Files Containing Juicy Info
intitle:index of apache.log | apache.logs,Files Containing Juicy Info
site:drive.google.com *.pdf,Files Containing Juicy Info
intitle:index of /key/ key.txt,Files Containing Juicy Info
inurl:linkedin.com view my resume facebook,Files Containing Juicy Info
intitle:index of .log,Files Containing Juicy Info
intitle:index of sysinfo,Files Containing Juicy Info
intitle:index of .exe,Files Containing Juicy Info
intitle:index of API*.txt,Files Containing Juicy Info
intitle:index of site:gov.np,Files Containing Juicy Info
intitle:index of *.mp4,Files Containing Juicy Info
intitle:index of admin*.txt,Files Containing Juicy Info
site:.nic.in inurl:.php?id=,Files Containing Juicy Info
inurl:.org intitle:index.of inflation,Files Containing Juicy Info
site:*/admin-portal/,Files Containing Juicy Info
intitle:index of site:gov.ru,Files Containing Juicy Info
intitle:index of site:gov.gr,Files Containing Juicy Info
site:.in | .com | .net intitle:index of ftp,Files Containing Juicy Info
inurl:forgotpassword.php,Files Containing Juicy Info
intitle:index of site:gov.*,Files Containing Juicy Info
intitle:index of /public_html,Files Containing Juicy Info
inurl:node_modules/ua-parser-js,Files Containing Juicy Info
intitle:index of /public/js,Files Containing Juicy Info
inurl:pastebin SHODAN_API_KEY,Files Containing Juicy Info
inurl:*gov intitle:index of/documents,Files Containing Juicy Info
inurl:.php?=*php site:.nic.in,Files Containing Juicy Info
intitle:index of /students,Files Containing Juicy Info
site:com rfp filetype:pdf,Files Containing Juicy Info
site:.edu intext:index of logs,Files Containing Juicy Info
intext:Index of /chatlogs,Files Containing Juicy Info
inurl:pastebin CVV,Files Containing Juicy Info
site: com intext:organisation data filetype:xls,Files Containing Juicy Info
intitle:index of default.asp,Files Containing Juicy Info
intitle:index of fileadmin,Files Containing Juicy Info
intitle:index of YaBB.pl,Files Containing Juicy Info
intitle:index of htsearch,Files Containing Juicy Info
intitle:index of glimpse,Files Containing Juicy Info
intitle:index of webdriver,Files Containing Juicy Info
intitle:index of index.php.bak,Files Containing Juicy Info
intitle:index of sendmail.inc,Files Containing Juicy Info
intitle:index of login.jsp,Files Containing Juicy Info
intitle:index of mod_auth_mysql,Files Containing Juicy Info
intitle:index of test.bat,Files Containing Juicy Info
intitle:index of msadcs.dll,Files Containing Juicy Info
intitle:index of browser.inc,Files Containing Juicy Info
intitle:index of hello.bat,Files Containing Juicy Info
intitle:index of dvwssr.dll,Files Containing Juicy Info
intitle:index of Servlet,Files Containing Juicy Info
intitle:index of upload.asp,Files Containing Juicy Info
inurl:pastebin API_KEY,Files Containing Juicy Info
inurl:pastebin Windows 10 Product Keys*,Files Containing Juicy Info
intitle:index of data*,Files Containing Juicy Info
intitle:index of document*.pdf,Files Containing Juicy Info
{intitle: indexof/.git },Files Containing Juicy Info
site:gov.hk intitle:index of /,Files Containing Juicy Info
inurl:pastebin AWS_ACCESS_KEY,Files Containing Juicy Info
site:*/forgotpassword.php,Files Containing Juicy Info
site:.edu intitle:index of,Files Containing Juicy Info
site:pastebin.com *@gmail.com password,Files Containing Juicy Info
site:.edu inurl:search,Files Containing Juicy Info
intitle:Index of DCIM/camera,Files Containing Juicy Info
intitle:Index of Screenshot,Files Containing Juicy Info
intitle:Index of system32,Files Containing Juicy Info
intitle:Index of Program files,Files Containing Juicy Info
intitle:Index of *.py,Files Containing Juicy Info
intitle:index of certificates,Files Containing Juicy Info
intitle:index of /.cpanel,Files Containing Juicy Info
index of :excel documents,Files Containing Juicy Info
intitle:index of :mobile number,Files Containing Juicy Info
intitle:index of node.js,Files Containing Juicy Info
intext:Index of intext:config.zip,Files Containing Juicy Info
inurl: conf/fastcgi.conf,Files Containing Juicy Info
inurl:conf/nginx.conf,Files Containing Juicy Info
site:com intitle:index of .env,Files Containing Juicy Info
intitle:Index of *.xlsx,Files Containing Juicy Info
inurl:/mutillidae/ Toggle Hints,Files Containing Juicy Info
intext:index of inurl:/etc/,Files Containing Juicy Info
inurl:wp-content/uploads/wooccm_uploads,Files Containing Juicy Info
intitle:index of particle.js,Files Containing Juicy Info
index of: invoice upload,Files Containing Juicy Info
intitle:index of Hindi movies,Files Containing Juicy Info
intext:index of wp-uploads,Files Containing Juicy Info
intext:index of signin,Files Containing Juicy Info
index of: marksheet upload,Files Containing Juicy Info
inurl:gov.uk,Files Containing Juicy Info
intext:Index of intext:users.zip,Files Containing Juicy Info
intext:Index of services.php | pass.php | passwd.php | credentials.txt,Files Containing Juicy Info
intitle:index of dhcp,Files Containing Juicy Info
index of:blog upload,Files Containing Juicy Info
inurl:cache/uploads,Files Containing Juicy Info
intitle:index of Apache/2.4.41 (Ubuntu) Server,Files Containing Juicy Info
index of: participants uploads,Files Containing Juicy Info
filetype:txt site:gitlab.* secret OR authtoken,Files Containing Juicy Info
site:gitlab.* intext:password intext:@gmail.com | @yahoo.com | @hotmail.com,Files Containing Juicy Info
inurl: */.env,Files Containing Juicy Info
intitle:index of /.git/config,Files Containing Juicy Info
intitle:index of */ftp.txt,Files Containing Juicy Info
intext:index of user-config,Files Containing Juicy Info
intitle:database backup filetype:sql,Files Containing Juicy Info
intext:sitemap filetype:txt,Files Containing Juicy Info
intext:pass filetype:txt,Files Containing Juicy Info
inurl:/package.json,Files Containing Juicy Info
intitle:index of username password filetype: xlsx,Files Containing Juicy Info
intitle:Index of /logs/ nginx,Files Containing Juicy Info
intext:index of home_page,Files Containing Juicy Info
intext:password | passwd | pwd site:anonfiles.com,Files Containing Juicy Info
site:*.example.com inurl:(elmah.axd | errorlog.axd) ext:axd,This dork can be used to identify public elmah instances which provides access to information about requests and responses, Session cookies, Session state, Query string and post variables, Physical path of the requested file of the application.
inurl:errorlog.axd ext:axd,This dork can be used to identify public elmah instances which provides access to information about requests and responses, Session cookies, Session state, Query string and post variables, Physical path of the requested file of the application.
showing putty logs,Files Containing Juicy Info
intitle:index of script.js,Files Containing Juicy Info
intitle:index of admin-config,Files Containing Juicy Info
intitle:index of admin.login.php,Files Containing Juicy Info
intitle:index of wp-mail-smtp,Files Containing Juicy Info
intitle:index of /resources,Files Containing Juicy Info
intext:index of ftp,Files Containing Juicy Info
intitle:index of untitled,Files Containing Juicy Info
intitle:index of untitled wp-content intext:scanned,Files Containing Juicy Info
index of :uploads parent salary intext:salary,Files Containing Juicy Info
index of :wp-config.zip,Files Containing Juicy Info
intitle:index of .ssh/authorized_keys,Files Containing Juicy Info
Intitle:database ext:sql,Files Containing Juicy Info
index of: parent directory uploads,Files Containing Juicy Info
index of: confidential uploads,Files Containing Juicy Info
index of: cache uploads,Files Containing Juicy Info
index of: QRcodes uploads,Files Containing Juicy Info
index of: contracts uploads,Files Containing Juicy Info
index of : phonebook,Files Containing Juicy Info
index of : truecaller uploads,Files Containing Juicy Info
index of: license upload,Files Containing Juicy Info
index of: certificate upload,Files Containing Juicy Info
index of: certificate wp-content,Files Containing Juicy Info
index of: application upload,Files Containing Juicy Info
index of: application form upload,Files Containing Juicy Info
index of: documents wp-content,Files Containing Juicy Info
intitle:index of _vti_inf.html,Files Containing Juicy Info
intitle:index of service.pwd,Files Containing Juicy Info
intitle:index of shtml.dll,Files Containing Juicy Info
inurl:admin ext:sql,Files Containing Juicy Info
index of:password wp-content,Files Containing Juicy Info
index of: putty uploads,Files Containing Juicy Info
intitle:index of master01,Files Containing Juicy Info
intitle:index of unidecode,Files Containing Juicy Info
intitle:index of cldr-data,Files Containing Juicy Info
intitle:index of gettext,Files Containing Juicy Info
intitle:index of src,Files Containing Juicy Info
intitle:index of src.hint,Files Containing Juicy Info
intitle:index of tar.xz,Files Containing Juicy Info
intitle:index of pkgs,Files Containing Juicy Info
intitle:index of ftp.riken,Files Containing Juicy Info
intitle:index of pub,Files Containing Juicy Info
intitle:index of cygwin,Files Containing Juicy Info
intitle:index of kde-l10n-de,Files Containing Juicy Info
intitle:index of txdot,Files Containing Juicy Info
intitle:index of mirror.koddos.net,Files Containing Juicy Info
intitle:index of Squid-cache,Files Containing Juicy Info
intitle:index of -login.php,Files Containing Juicy Info
intitle:index of metin,Files Containing Juicy Info
intitle:index of html-en,Files Containing Juicy Info
intitle:index of html-intro,Files Containing Juicy Info
intitle:index of echo-linux,Files Containing Juicy Info
intitle:index of filelist.xml,Files Containing Juicy Info
intitle:index of wp-content,Files Containing Juicy Info
intitle:index of css,Files Containing Juicy Info
intitle:index of CD.pdf,Files Containing Juicy Info
intitle:index of DOCS-TECH,Files Containing Juicy Info
intitle:index of Server-Side,Files Containing Juicy Info
intitle:index of py-text,Files Containing Juicy Info
Google Dork,Google Dork
intitle:index of htdocs,Files Containing Juicy Info
intitle:index of .ppt,Files Containing Juicy Info
site:github.com intext:unattend xmlns AND password ext:xml,Files Containing Juicy Info
intitle:index of workspace.xml,Files Containing Juicy Info
intitle:index of -qpf,Files Containing Juicy Info
intitle:index of -ipk,Files Containing Juicy Info
intitle:index of Packages.gz,Files Containing Juicy Info
intitle:index of mips32el-nf,Files Containing Juicy Info
intitle:index of .phpunit.xml,Files Containing Juicy Info
intitle:index of .AndroidManifest.xml,Files Containing Juicy Info
intitle:Index of / intext:pass.txt,Files Containing Juicy Info
inurl:WS_FTP.log,Files Contaning Juicy Info
intext:Index of email.txt,Files Containing Juicy Info
intitle:index of pptx,Files Containing Juicy Info
intitle:index of ppt.html,Files Containing Juicy Info
intitle:index of slides-ppt,Files Containing Juicy Info
intitle:index of -XML.pdf,Files Containing Juicy Info
intitle:index of XML,Files Containing Juicy Info
intitle:index of XML.Xerces,Files Containing Juicy Info
intitle:index of infn.it,Files Containing Juicy Info
intitle:index of lngs.infn.it,Files Containing Juicy Info
intitle:index of extra,Files Containing Juicy Info
intitle:index of extranet,Files Containing Juicy Info
intitle:index of fsi,Files Containing Juicy Info
intitle:index of oxid-esales,Files Containing Juicy Info
site:pastebin.com intext:license key | expiration,Files Containing Juicy Info
site:pastebin.com intext:username | password | secret_key | token,Files Containing Juicy Info
intitle:index.of /email /robots.txt,Files Containing Juicy Info
intitle:index.of /cftp /robots.txt,Files Containing Juicy Info
allinurl:index.php?page= site:.gov.in,Files Containing Juicy Info
inurl:php?id= site:.gov.bd,Files Containing Juicy Info
Index of /vendor/spatie/robots-txt,Files Containing Juicy Info
intitle:index of .private.xml,Files Containing Juicy Info
site:pastebin.com intext:administrator:500:,Files Containing Juicy Info
inurl:php?id= site:.com,Files Containing Juicy Info
Index of /apidoc/api-web/target/classes/,Files Containing Juicy Info
intitle:password reset,Files Containing Juicy Info
intitle:index.of /CMS /robots.txt,Files Containing Juicy Info
intitle:index of server.log,Files Containing Juicy Info
intitle:index of /backup/sql,Files Containing Juicy Info
intitle:index of cv site:.com,Files Containing Juicy Info
intext:swagger filetype:log,Files Containing Juicy Info
intitle:index of server.properties,Files Containing Juicy Info
inurl:robots | robot intext:admin AND Disallow ext:txt,Files Containing Juicy Info
intitle:index of mongod*,Files Containing Juicy Info
intitle:index.of wp.login,Files Containing Juicy Info
inurl:/wp-content/plugins/simple-forum/admin/,Files Containing Juicy Info
intitle:index.of /Snowflake /robots.txt,Files Containing Juicy Info
intitle:index of .env.example,Files Containing Juicy Info
inurl:app.yaml intext:runtime: ext:yaml,Files Containing Juicy Info
inurl: https://app.zerocopter.com/rd/,Files Containing Juicy Info
intitle:index.of conf.mysql,Files Containing Juicy Info
intext:password intitle:index of,Files Containing Juicy Info
Fwd: intitle:Authorize application Learn more about OAuth,Files Containing Juicy Info
inurl:/wp-content/plugins/elementor/,Files Containing Juicy Info
inurl:/wp-content/plugins/wp-filebase/,Files Containing Juicy Info
intitle:index of console,Files Containing Juicy Info
intitle:index of logs,Files Containing Juicy Info
index of / inurl:/pki/,Files Containing Juicy Info
intext:index of/ top secret gov,Files Containing Juicy Info
inurl:/servicedesk/customer/user/signup,Files Containing Juicy Info
inurl:wp-content/plugins/easy-wp-smtp,Files Containing Juicy Info
Re: inurl:/app/kibana#,Files Containing Juicy Info
intext:adobe coldfusion 8,Files Containing Juicy Info
New Dork,Files Containing Juicy Info
intitle:Index of inurl:data/plugins/,Files Containing Juicy Info
intitle:Index of Apache/2.4.50,Files Containing Juicy Info
site:*/node_modules/ content:ssh,Files Containing Juicy Info
`;

        document.addEventListener('DOMContentLoaded', () => {
            const dorksGrid = document.getElementById('dorks-grid');
            const searchInput = document.getElementById('search-input');
            const categoryFilter = document.getElementById('category-filter');
            const resultsCount = document.getElementById('results-count');
            const noResults = document.getElementById('no-results');
            const toast = document.getElementById('toast');
            let allDorks = [];
            let categories = {};

            function categorizeDork(dork) {
                const d = dork.toLowerCase();
                if (d.includes('password') || d.includes('api_key') || d.includes('secret') || d.includes('shodan_api_key') || d.includes('credentials') || d.includes('passwd') || d.includes('key.txt')) return 'Sensitive Information';
                if (d.includes('login') || d.includes('signin') || d.includes('cpanel') || d.includes('portal') || d.includes('signup') || d.includes('forgotpassword')) return 'Login Portals';
                if (d.includes('.env') || d.includes('settings.py') || d.includes('.config') || d.includes('.xml') || d.includes('docker-compose') || d.includes('.yaml')) return 'Config & Env Files';
                if (d.includes('intitle:index of') || d.includes('intext:index of')) return 'Directory Listings';
                if (d.includes('filetype:') || d.includes('ext:')) return 'Specific Filetypes';
                if (d.includes('site:gov') || d.includes('site:.edu') || d.includes('site:.nic.in')) return 'Gov/Edu Targets';
                if (d.includes('wp-content') || d.includes('wordpress') || d.includes('wp-mail-smtp')) return 'WordPress Specific';
                if (d.includes('.log') || d.includes('logs')) return 'Log Files';
                if (d.includes('sql') || d.includes('database') || d.includes('mysql')) return 'Database Related';
                return 'Miscellaneous';
            }

            function processDorks() {
                const uniqueDorks = new Set();
                allDorks = rawDorksData
                    .split('\n')
                    .filter(line => line.trim() !== '')
                    .map(line => {
                        const parts = line.split(',');
                        const dork = parts[0].trim();
                        const description = parts.slice(1).join(',').trim() || "No description available.";
                        return { dork, description };
                    })
                    .filter(item => {
                        if (!item.dork || item.dork.toLowerCase() === 'google dork' || item.dork.toLowerCase() === 'new dork' || uniqueDorks.has(item.dork.toLowerCase())) {
                            return false;
                        }
                        uniqueDorks.add(item.dork.toLowerCase());
                        return true;
                    })
                    .map(item => {
                        const category = categorizeDork(item.dork);
                        return { ...item, category };
                    });

                categories = allDorks.reduce((acc, curr) => {
                    acc[curr.category] = (acc[curr.category] || 0) + 1;
                    return acc;
                }, {});

                const sortedCategories = Object.keys(categories).sort();
                sortedCategories.forEach(cat => {
                    const option = document.createElement('option');
                    option.value = cat;
                    option.textContent = `${cat} (${categories[cat]})`;
                    categoryFilter.appendChild(option);
                });
            }

            function renderDorks(dorksToRender) {
                dorksGrid.innerHTML = '';
                if(dorksToRender.length === 0){
                    noResults.classList.remove('hidden');
                } else {
                    noResults.classList.add('hidden');
                }
                
                resultsCount.textContent = `Showing ${dorksToRender.length} of ${allDorks.length} dorks.`;

                dorksToRender.forEach((item, index) => {
                    const card = document.createElement('div');
                    card.className = 'bg-white rounded-xl shadow-lg p-5 flex flex-col justify-between border border-transparent hover:border-blue-500 hover:shadow-2xl transition-all duration-300 card-enter';
                    
                    const categoryColorClasses = {
                        'Sensitive Information': 'bg-red-100 text-red-800',
                        'Login Portals': 'bg-yellow-100 text-yellow-800',
                        'Config & Env Files': 'bg-purple-100 text-purple-800',
                        'Directory Listings': 'bg-blue-100 text-blue-800',
                        'Specific Filetypes': 'bg-green-100 text-green-800',
                        'Gov/Edu Targets': 'bg-indigo-100 text-indigo-800',
                        'WordPress Specific': 'bg-sky-100 text-sky-800',
                        'Log Files': 'bg-gray-100 text-gray-800',
                        'Database Related': 'bg-orange-100 text-orange-800',
                        'Miscellaneous': 'bg-pink-100 text-pink-800',
                    };
                    
                    const categoryClasses = categoryColorClasses[item.category] || 'bg-gray-100 text-gray-800';

                    card.innerHTML = `
                        <div>
                            <p class="text-xs font-semibold uppercase ${categoryClasses} inline-block px-2 py-1 rounded-full mb-3">${item.category}</p>
                            <p class="font-mono bg-gray-100 p-3 rounded-md text-gray-700 text-sm break-words">${item.dork}</p>
                            <p class="text-gray-600 mt-3 text-sm">${item.description}</p>
                        </div>
                        <div class="flex space-x-2 mt-4">
                            <button data-dork="${item.dork}" class="copy-btn flex-1 bg-blue-500 text-white py-2 px-3 rounded-lg hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 transition-transform transform hover:scale-105 text-sm flex items-center justify-center">
                                <i class="fas fa-copy mr-2"></i>Copy
                            </button>
                            <a href="https://www.google.com/search?q=${encodeURIComponent(item.dork)}" target="_blank" class="test-btn flex-1 bg-gray-700 text-white py-2 px-3 rounded-lg hover:bg-gray-800 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500 transition-transform transform hover:scale-105 text-sm flex items-center justify-center">
                                <i class="fas fa-external-link-alt mr-2"></i>Test
                            </a>
                        </div>
                    `;
                     setTimeout(() => {
                        card.classList.add('card-enter-active');
                    }, index * 20);
                    dorksGrid.appendChild(card);
                });
            }
            
            function showToast() {
                toast.classList.remove('opacity-0', 'translate-y-2');
                toast.classList.add('opacity-100', 'translate-y-0');
                setTimeout(() => {
                    toast.classList.remove('opacity-100', 'translate-y-0');
                    toast.classList.add('opacity-0', 'translate-y-2');
                }, 2000);
            }

            function filterAndRender() {
                const searchTerm = searchInput.value.toLowerCase();
                const selectedCategory = categoryFilter.value;

                const filteredDorks = allDorks.filter(item => {
                    const matchesSearch = item.dork.toLowerCase().includes(searchTerm) || item.description.toLowerCase().includes(searchTerm) || item.category.toLowerCase().includes(searchTerm);
                    const matchesCategory = selectedCategory === 'all' || item.category === selectedCategory;
                    return matchesSearch && matchesCategory;
                });
                renderDorks(filteredDorks);
            }
            
            function renderChart(){
                const ctx = document.getElementById('dorksChart').getContext('2d');
                const sortedCategories = Object.entries(categories).sort(([,a],[,b]) => b-a);
                const chartLabels = sortedCategories.map(c => c[0]);
                const chartData = sortedCategories.map(c => c[1]);
                
                new Chart(ctx, {
                    type: 'bar',
                    data: {
                        labels: chartLabels,
                        datasets: [{
                            label: 'Number of Dorks',
                            data: chartData,
                            backgroundColor: [
                                'rgba(54, 162, 235, 0.6)',
                                'rgba(255, 99, 132, 0.6)',
                                'rgba(255, 206, 86, 0.6)',
                                'rgba(75, 192, 192, 0.6)',
                                'rgba(153, 102, 255, 0.6)',
                                'rgba(255, 159, 64, 0.6)',
                                'rgba(199, 199, 199, 0.6)',
                                'rgba(83, 102, 255, 0.6)',
                                'rgba(100, 255, 64, 0.6)',
                                'rgba(255, 99, 255, 0.6)'
                            ],
                            borderColor: [
                                'rgba(54, 162, 235, 1)',
                                'rgba(255, 99, 132, 1)',
                                'rgba(255, 206, 86, 1)',
                                'rgba(75, 192, 192, 1)',
                                'rgba(153, 102, 255, 1)',
                                'rgba(255, 159, 64, 1)',
                                'rgba(199, 199, 199, 1)',
                                'rgba(83, 102, 255, 1)',
                                'rgba(100, 255, 64, 1)',
                                'rgba(255, 99, 255, 1)'
                            ],
                            borderWidth: 1
                        }]
                    },
                    options: {
                        responsive: true,
                        maintainAspectRatio: false,
                        indexAxis: 'y',
                        scales: {
                            y: {
                                beginAtZero: true,
                                ticks: {
                                    font: {
                                        size: 14
                                    }
                                }
                            },
                            x: {
                                ticks: {
                                    font: {
                                        size: 14
                                    }
                                }
                            }
                        },
                        plugins: {
                            legend: {
                                display: false
                            },
                            tooltip: {
                                callbacks: {
                                    label: function(context) {
                                        return ` ${context.dataset.label}: ${context.raw}`;
                                    }
                                }
                            }
                        }
                    }
                });
            }

            processDorks();
            renderDorks(allDorks);
            renderChart();

            searchInput.addEventListener('input', filterAndRender);
            categoryFilter.addEventListener('change', filterAndRender);
            
            dorksGrid.addEventListener('click', (e) => {
                const button = e.target.closest('.copy-btn');
                if (button) {
                    const dorkToCopy = button.dataset.dork;
                    navigator.clipboard.writeText(dorkToCopy).then(() => {
                        showToast();
                    }).catch(err => {
                        console.error('Failed to copy: ', err);
                    });
                }
            });
        });
    </script>
</body>
</html>
