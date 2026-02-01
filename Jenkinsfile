// node {
//     def appDir = '/var/www/nextjs-app'

//     stage('Clean Workspace'){
//         echo 'Cleaning Jenkins Workspace'
//         deleteDir()
//     }

//     stage('Clone Repo'){
//         echo 'Cloning the repo'
//         git(
//             branch: 'main',
//             url: 'https://github.com/adnanshafique468/jenkins-app'
//         )
//     }

//     stage('Deploy to EC2'){
//         echo 'Deploying to EC2'
//         sh """
//             sudo mkdir -p ${appDir}
//             sudo chown -R jenkins:jenkins ${appDir}

//             rsync -av --delete --exclude='.git' --exclude='node_modules' ./ ${appDir}

//             cd ${appDir}
//             sudo npm install
//             sudo npm run build

//             sudo fuser -k 3000/tcp || true
//             npm run start
//         """
//     }
// }    

node {
    def appDir = '/var/www/nextjs-app'

    stage('Clean Workspace') {
        echo 'Cleaning Jenkins Workspace'
        deleteDir()
    }

    stage('Clone Repo') {
        echo 'Cloning the repo'
        git(
            branch: 'main',
            url: 'https://github.com/adnanshafique468/cicd-jenkins'
        )
    }

    stage('Setup Node & Tailwind') {
        echo 'Setting up Node 20 and Tailwind PostCSS plugin'
        sh """
            # Install Node.js 20 using NodeSource (Ubuntu)
            curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
            sudo apt install -y nodejs build-essential

            # Verify Node & npm
            node -v
            npm -v

            # Install @tailwindcss/postcss in project
            cd ${appDir}
            npm install -D @tailwindcss/postcss
        """
    }

    stage('Deploy to EC2') {
        echo 'Deploying to EC2'
        sh """
            # Create app directory if not exists
            sudo mkdir -p ${appDir}
            sudo chown -R jenkins:jenkins ${appDir}

            # Sync project files (ignore .git & node_modules)
            rsync -av --delete --exclude='.git' --exclude='node_modules' ./ ${appDir}

            # Build Next.js app
            cd ${appDir}
            npm install
            npm run build

            # Kill old process if running on port 3000
            sudo fuser -k 3000/tcp || true

            # Start app using PM2 (background)
            pm2 stop nextjs-app || true
            pm2 start npm --name "nextjs-app" -- run start
            pm2 save
        """
    }
}
